# DB Guide

This document describes the design and operation of the RebDB Pseudo-Relational database.

## Introduction

RebDB is a simple yet powerful SQL-like database written entirely in Rebol/Base compatible syntax. The text and examples in this guide assume some familiarity with both Rebol and RDBMS's in general.

### Distribution Scripts

Script      | Description
----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
db.r        | Defines the main workings of RebDB in a single context named `db`. A number of accessor functions, all prefixed with `db-`, are exported to the global context for easy reference.
db-client.r | Implements the `db-request` function used by remote clients, and provides the `sql` function for easy database access.
SQL.r       | Starts up a console SQL client.
login.sql   | Used by the SQL client to initialize various settings.

## Storage Architecture

A number of discrete components make up the RebDB storage architecture. Understanding these will enable you to get the most out of your database.

### Tables

A table consists of a control file (with a `.ctl` suffix), which describes the organization of its data, and a data file (with a `.dat` suffix) which contains the actual data in Rebol native format.

### Control Files

Control files contain the plain text definition of a table. As an example:

	Columns [ID Date Note]

This defines a table with three columns: ID, Date and Note.

### Data Files

At the heart of the database are data-files. When a table is first accessed its data-file is loaded into memory.
If you are working with exceedingly large files, or very limited memory, RebDB may not be suitable for you.

### Log Files

Every statement that results in rows being changed is logged to the effected table's log file. If RebDB terminates prior to a commit you can "replay" the missing transactions by running the log file (which is just an SQL script).

### Direct Load

If you need to load large volumes of data into RebDB you should do this directly via Rebol.

First, create a table for your data.

	db-create my-table [ID Date Note]

Now create a block to hold your data.

	blk: make block! 300000 ; rows * cols

Then write a simple loop that appends your values to this block.

	repeat i 100000 [
	    insert tail blk reduce [i 1-Jan-2004 - to-integer i / 100 join "Note-" i]
	]

Now we just need to write the data to disk and verify that RebDB likes it.

	save/all %my-table.dat sort/skip/all blk 3 ; sort all cols
	db-desc my-table

While this technique is suitable for creating large tables, periodic inserts should use the `db-insert` function as it guarantees data integrity.

### Next Value

The `next` token in an insert statement causes the token to be replaced by the last value in the column plus one. The following code illustrates its use.

	db-insert my-table [next 1-Jan-2000 "Note"]

The value defaults to 1 if this is the first row.

Similar to *sequences* and *auto-increment* facilities in other databases, this allows you to easily create sequenced record numbers so long as care is taken when updates are performed.

### Sorted / Dirty Flags

When RebDB opens a table it sets the table's `sorted?` flag to **true** and the table's `dirty?` flag to **false**. Subsequent operations may change the values of these flags.

Operation   | Sorted flag set to ...                                 | Dirty flag set to ...
----------- | ------------------------------------------------------ | -----------------------------
sort-table  | True (used by lookups, key searches & commits)         | NA
db-commit   | NA                                                     | False
db-delete   | NA                                                     | True if any rows were deleted
db-insert   | True if new row is >= last row (or the table is empty) | True
db-truncate | True                                                   | True
db-update   | False if any rows were updated                         | True if any rows were updated

You can determine the current values of these flags by using `db-tables`.

### Commit & Rollback

A commit does the following:

1. Sorts the table if required,
2. Saves the table to disk,
3. Sets the dirty flag to false,
4. Deletes the log file.

A rollback does the following:

1. Deletes the log file,
2. Closes the table.

Trying to open a table with an uncommitted log file will result in an error. Either delete the log file or apply its changes to the table.

### Storage Tuning

The following points should be noted when creating new tables.

* As each value requires a minimum of two (2) bytes to store, ensure that your tables have as few columns and rows as necessary.
* Avoid storing large values such as `image!` or `binary!` directly. Instead, consider storing a `file!` reference to them.
* Avoid operations that cause "thrashing" such as a tight insert-lookup loop where the inserted row is less than the last row, or a tight insert-commit loop.

## Search Engine

A database needs to be able to retrieve data, even if only to subsequently delete or update it, to be of any use. The search engine is a collective term for the lookup, search and fetch processes that enable this.

### Lookups

A lookup accepts a key of one or more values and conducts a binary search until it finds the first row that matches the key. The match is done in the same order as the columns of the table (i.e. A key of `[1 "Bob"]` will find the first row that has a value of 1 in the first column of the table and a value of "Bob" in the second).

### Searches

#### Rowid

A rowid search is triggered when the first token of the where block is the word `rowid`. Rowid searches are optimized for equality ("=") and range checking ("<", ">" and "between"). For example:

	SQL> select * from my-table where [rowid = 1]
	SQL> select * from my-table where [rowid < 5]
	SQL> select * from my-table where [rowid > 3]
	SQL> select * from my-table where [rowid between 3 5]

The `last` token can be used and is replaced by the number of rows in the table. Examples of its use are:

	SQL> select * from my-table where [rowid = last]
	SQL> select * from my-table where [rowid between 3 last]

There is no matching *first* token as the first rowid will always be 1.

#### Key

A key search is triggered when the where clause specifies a key of one or more values and behaves much like a lookup except that all rows matching the key are returned. For example:

	SQL> select * from my-table where 1
	SQL> select * from my-table where [1 10-Jan-2004]

Key searches are highly optimized in that they rely on a sorted table to find the first and last rows matching the key and then copy all intervening rows. A "*" key search is more efficient than selecting specific columns.

#### Equality

An equality search on a single column that is not the first (i.e. not a key search as above) uses find to quickly scan the column for a match. For example:

	SQL> select * from my-table where [date = 10-Jan-2004]

#### Linear

When no other option is available every row is read in turn and compared to the predicate. The speed of this operation depends on the complexity of the predicate and the size of the table. For example:

	SQL> select * from my-table where [id < 5]

### Fetches

A fetch occurs when a select statement is issued with no where clause. A "*" fetch is more efficient than selecting specific columns. For example:

	SQL> select * from my-table
	SQL> select id from my-table
	SQL> select [id date] from my-table

### Search Efficiency

The search engine takes advantage of the fact that all data is:

1. In Rebol format,
2. In memory,
3. Always sorted.

The search paths, in order of efficiency, are as follows.

Type     | Description              | Statement
-------- | ------------------------ | -----------------------------------------------------------------------
Binary   | Match a unique key       | lookup table 1
Rowid    | Match rowid(s)           | select * from table where [rowid = 1]
Key *    | Retrieve matching row(s) | select * from table where 1<br>select * from table where [col1 = 1]
Key      | Retrieve matching row(s) | select col from table where 1<br>select col from table where [col1 = 1]
Fetch *  | Fetch all columns        | select * from table
Fetch    | Fetch column(s)          | select col from table
Equality | Find a value             | select * from table where [col2 = 1]
Linear   | Compare all rows         | select * from table where [col1 < 2]

### Statement Handling

An SQL statement entered at the prompt of an SQL client is "handled" at three distinct levels; first by the client itself, then by the `sql` function, and finally by the underlying `db-` function. This process is depicted below.

#### SQL.r

	SQL> select * from my-table where [id < 5] order by id desc

#### db-client.r

	sql [select * from my-table where [id < 5] order by id desc]

#### db.r

	db-select/where/order/desc * my-table [id < 5] id

### Search Tuning

Getting results back quickly takes more than just selecting the fewest possible columns of the fewest possible rows. The following are general observations.

* Use `lookup` in preference to `select`.
* Use key value(s) in preference to a predicate.
* Use a single check for equality in preference to other predicate conditions.
* Use `rowid` when possible.
* Have as few conditions in the predicate as possible.
* Retrieve one column in preference to "*".
* Retrieve "*" in preference to other combinations / order of columns.
* Avoid unnecessary sorts, and use as few sort columns as possible.

## Specific Features

This section covers some specific features of RebDB that require more explanation than that found in the SQL Guide.

### Aggregate Functions

#### Distinct

The `distinct` refinement sorts the result set and removes all duplicate rows. It is used as follows:

	SQL> select distinct * from my-table
	SQL> select distinct [id date] from my-table

#### Count, Min, Max, Sum, Avg and Std

These aggregate functions return the indicated aggregation of a column. They are used as follows:

	SQL> select count id from my-table
	SQL> select min id from my-table
	SQL> select max id from my-table
	SQL> select sum id from my-table
	SQL> select avg id from my-table
	SQL> select std id from my-table

#### Group By

If more than one column appears in the select list then all columns prior to the first must be "grouped", with the last column in the select list being the one that will be aggregated. Examples of this are:

	SQL> select count [date id] from my-table group by date
	SQL> select min [date note id] from my-table group by [note date]

#### Having

A `having` clause is much like a `where` clause except it is applied to the results of a *grouped* aggregate, as in:

	SQL> select count [date id] from my-table group by date having [count > 2]

Note that the above example is saying, "where the number of ids for a given date exceed two", not, "where the id is greater than two".

### rowid Pseudo-column

In addition to appearing in a predicate, `rowid` may be used as a pseudo-column in select statements.

	SQL> select rowid from my-table
	SQL> select [date rowid] from my-table order by rowid

It is commonly used in a *cursor*, as in the following code:

	cursor: db-select rowid my-table
	foreach rowid cursor [
	    print reform [
	        db-select/where * my-table compose [rowid = (rowid)]
	    ]
	]

### Star "*" Expansion

Four statements accept the star "*" expansion character.

Statement              | Description
---------------------- | ------------------------------------------------------------------------------------------------------------
select * from my-table | Replaces the "*" with all the table's column names in the same order they are specified in the control file.
close *                | Closes all open tables.
commit *               | Commits all open tables with changes pending.
rollback *             | Rolls back all open tables with changes pending.

### Soundex Function

The `soundex` function can be used in predicates as follows:

	SQL> select * from my-table where [(soundex "Smith") = soundex note]

And will match strings that are phonetically similar (as in "Smith" and "Smyth").
As soundex returns a four-byte string, you can dramatically improve these operations by storing soundexed values in the database. Just make sure that they are updated along with the column value(s) they are based on!

### Explain

The explain function, typically used with select, will return a statement trace in lieu of results.

	explain [sql [select * from my-table]]
	explain [db-select * from my-table]

The SQL client, depicted below, expects the `explain` token to precede the statement as in the following example:

	SQL> explain select * from my-table

	Seq Step  Type     Time
	--- ----- -------- ----
	1   Open  my-table 0:00
	2   Fetch *        0:00

	2 row(s) selected in 0:00 seconds
