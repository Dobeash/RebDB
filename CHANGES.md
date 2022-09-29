# 2.0.3 - September 2008

This release corrects a flaw with searching for stored block values, and simplifies the `db-commit` code. In addition, a number of enhancements have been made.

## Updated SQL function

The `sql` function now accepts strings, so:

	sql [select * from my-table]

can now also be written as:

	sql "select * from my-table"

The `sql` function now also performs colon (":") argument substitution, allowing statements like the following:

	sql ["select * from my-table where [id = :1]" 1]

## Master/detail joins

The `db-select` function has been enhanced to accept statements in these additional forms:

	select * from master joins [select * from details where :id] on id
	select * from master joins [select * from details where [all [master-id = :id master-date = :date]] on [id date]

which works exactly like a normal join with the following differences:

* It can only join one table to another,
* Detail columns are always joined to the right of master columns,
* Table.column prefixes are not supported so all columns in the join must be uniquely named.

## List of Values lookups

The `db-select` function has been enhanced to accept statements in these additional forms:

	select * from table replaces id with name
	select * from table replaces [id-1 id-2] with [table-1 table-2]

which performs a highly optimized db-lookup for each replaced value, but has the following restrictions:

* The lookup expects lookup tables in the form `[id label other-column(s)]`,
* Only single-key lookups are supported,
* A lookup that fails will replace the column value with `none!`.

## /joins and /replaces examples

Let's create some tables:

	sql "create table Customers [ID Name Address]"
	sql "create table Orders [Customer-ID Date Order-ID]"
	sql "create table Items [ID Description Price Quantity Total]"

then populate them with some test data:

	repeat i 100 [
	    sql reduce ["insert into Customers values [:1 {:2} {:3}]" i join "Name " i join "Address " i]
	]

	p: 0
	repeat i 100 [
	    repeat j 10 [
	        sql reduce ["insert into Orders values [:1 :2 :3]" i now/date + i p: p + 1]
	    ]
	]

	repeat i 10000 [
	    sql reduce ["insert into Items values [:1 {:2} $1.99 1 $1.99]" i join "Description " i]
	]

Now for some joins:

	sql "select * from Orders where 1 joins [select * from Items where :Order-ID] on Order-ID"
	sql "select * from Orders joins [select * from Items where :Order-ID] on Order-ID"

and a replace:

	sql "select * from Orders replaces Customer-ID with Customers"

# 2.0.2

This release corrects a critical flaw with multi-column `distinct` use.

## DISTINCT clause with multiple columns

Due to the way in which Rebol handles unique/skip, RebDB `select` statements that used the `distinct` clause and returned more than one column may have returned a subset of the correct rows. The following Rebol code demonstrates this problem:

	>> unique/skip [1 "A" 1 "B"] 2
	== [1 "A"]
	>> unique/skip ["A" 1 "B" 1] 2
	== ["A" 1 "B" 1]

RebDB v2.0.2 no longer uses `unique/skip`.

## Compatibility

RebDB now requires Core 2.6.0 (included with View 1.3) or greater.

## Lines

By default, RebDB data files are now formatted with one row of data per line. You can revert to the old behaviour by setting `db/lines?: false`, or using the new `set lines` option from the RebDB client (see below).

## SQL client set options

### Browser

The `set browser` option has been removed as the Rebol `set-browser-path` function has been depreciated.

### Lines

RebDB's line mode can now be toggled on or off with the `set lines` option.

### Path

RebDB's base data directory can now be set/changed with the `set path` option.

### Help

The `help` command has been updated to reflect the above changes.

# 2.0.1

This release is a maintenance release with one major enhancement: automatic log recovery.

## Automatic log recovery

RebDB now automatically applies all log files at startup. You can see this behaviour for yourself by issuing the following statements:

	SQL> create test [col]
	SQL> insert into test values [0]
	SQL> exit

and then restart the SQL client to see the following replay messages:

	Replaying 1 change(s).
	insert into test values [0]
	Replay completed.

## SQL client error messages

Underlying Rebol errors (e.g. math overflow) are now trapped and displayed correctly. The following statements demonstrate this:

	SQL> create test [col]
	SQL> insert into test values [2000000000]
	SQL> insert into test values [2000000000]
	SQL> select sum col from test
