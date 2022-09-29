# SQL Guide

## Introduction

This document provides a brief introduction to the Structured Query Language (SQL) used to access the database. Those familiar with MySQL, Oracle, DB2, MS*Access or SQL Server (amongst others) should note that only a subset of the full ANSI-standard SQL operations are supported.

## SQL Statements

SQL statements consist of one or more space separated tokens terminated by a carriage return. In the following examples a plural indicates that one or more such space separated tokens may be provided (e.g. `values`) within opening and closing brackets.

Tokens seperated by a vertical bar ("|") indicate optional choices.

### Row Retrieval

#### Lookup

Returns first row that matches a key.

	lookup table key | [keys]

#### Select

Returns columns and rows from a table.

	select distinct | aggregate | * | column | [rowid | columns]
	from   table
	where  key | [keys] | [conditions]
	group by column | [columns] | having [conditions]
	order by column | [columns] | desc

### Row Management

#### Delete

Deletes row(s) from a table.

	delete from table key | [keys]
	delete from table where key | [keys] | [conditions]

#### Insert

Appends a row of values to a table.

	insert into table values [next | values]

#### Truncate

Deletes all rows from a table.

	truncate table

#### Update

Updates row(s) in a table.

	update table set column to value key | [keys]
	update table set column to value where key | [keys] | [conditions]
	update table set [columns] to [values] key | [keys]
	update table set [columns] to [values] where key | [keys] | [conditions]
	update table set [columns] to [expressions] key | [keys]
	update table set [columns] to [expressions] where | key | [keys] | [conditions]

### Table Management

These statements enable you to manage tables.

#### Close

Closes a table with no changes pending.

	close table | *

#### Commit

Saves a table with changes pending.

	commit table | *

#### Create

Creates a table.

	create table [columns]

#### Drop

Drops a table.

	drop table

#### Rollback

Closes a table with changes pending.

	rollback table | *

### Informational

These statements give you information about various data structures.

#### Describe

Information about the columns of a table.

	desc table
	describe table

#### Explain

Executes statement and returns plan.

	explain statement

Preceding a statement with explain causes the statement to be executed as per normal, then return with a statement trace in lieu of its normal result(s).

#### Rows

Number of rows in table.

	rows table

#### Show

Database statistics.

	show

#### Table?

Returns true if table exists.

	table? table

#### Tables

Information about currently open tables.

	tables

## Aggregation

### Distinct

The distinct clause causes the entire result set to be sorted with all duplicate rows removed. Some examples:

	select distinct * from table
	select distinct column from table

### Functions

The select statement supports a number of aggregation functions.

Function | Description                                           | Usage
-------- | ----------------------------------------------------- | ------------------------------
Count    | Number of values meeting criteria.                    | select count column from table
Min      | Minimum value meeting criteria.                       | select min column from table
Max      | Maximum value meeting criteria.                       | select max column from table
Sum      | Summation of values meeting criteria.                 | select sum column from table
Avg      | Arithmetic mean (average) of values meeting criteria. | select avg column from table
Std      | Standard deviation of values meeting criteria.        | select std col from table

### Group By

If multiple columns appear in the select list then all columns prior to the last must be grouped. Aggregate functions are always applied to the last column appearing in the select list. Some examples:

	select count [column1 column2] from table group by column1
	select count [column3 column1 column2] from table group by [column1 column3]

### Having

The having clause is similar to a where clause except it is applied to the aggregated column of a group by. Some examples:

	select count [col1 col2] from table group by col1 having [count > 20]
	select sum [col1 col2] from table group by col1 having [any [sum < 4 sum > 8]]

## Conditions

Condition    | Usage
------------ | -------------------------------------------------------------------
Arithmetic   | column {= <> < > <= >=} value
True / False | {zero? odd? even? none? empty?} column<br>{string? integer?} column
Search       | find column string<br>find [values] column
Not          | not condition
And / Or     | {any all} [conditions]

## Values

Value   | Usage
------- | ------------------------------------------------------
String  | A sequence of characters delimited with double quotes.
Integer | A whole number.
Decimal | A number with at least one decimal place.
Date    | A date in the form **DD-MON-YYYY**.
Money   | A dollars and cents amount.

## Expressions

Expression | Usage
---------- | ---------------------------------------------------------
Arithmetic | column {+ - / // **} value<br>column {+ - / // **} column
Positional | {first second third last} column<br>pick column position
String     | (uppercase lowercase} column<br>copy/part column length

## SQL Client

A SQL client is a software program that provides an environment in which you can interactively run scripts and execute statements against the database. The standard RebDB client is a simple console client where commands are entered at a prompt and results displayed below that. These *commands* fall into four groups as detailed below.

### Statements

An SQL statement, such as a select, which is sent to the database for processing.

### Commands

A command that is processed by the client. The following commands are supported.

Command | Description                                                                                  | Usage
------- | -------------------------------------------------------------------------------------------- | ----------------------------
Echo    | A line commencing with the echo command will display the string that follows it.             | SQL> echo "Have a nice day."
Exit    | Exit a client by clicking the close icon of the window or typing exit at the console prompt. | SQL> exit
Help    | Typing help will display a concise summary of available statements, commands and settings.   | SQL> help
Run     | Typing #run script# at the console prompt will run the nominated script.                     | SQL> run login.sql
Set     | Typing set followed by a setting and value will change the setting, while entering set by itself will display the current settings. | SQL> set username Bob

### Settings

Settings control how the client behaves, and are typically specified in a login.sql script (see later). All settings default to none.

Setting  | Description | Usage
-------- | ----------- | -----
Address | This setting determines what IP address and port number the client should send its statements to. If no IP address is provided, it determines what port number the client should listen on for incoming requests. This latter setting, used in a login.sql script, is how a *server* is launched. | set address tcp://127.0.0.1:1000<br>set address tcp://:1000
HTML    | This setting determines whether HTML output is on or off. The default file suffix, if not otherwise specified, is `.html`. | set html query.html<br>set html off<br>set html query
Spool   | This setting determines whether spool output is on or off. The default file suffix (if not one of `.txt`, `.lst` or `.log`) is `.txt`. | set spool session.txt<br>set spool off<br>set spool session
Username| This setting sets the username sent to a RebDB server, which can be useful in a multi-user environment when trying to determine who did what and when. | set username Bob

### Comments

A comment begins with a semi-colon (";") and can appear anywhere on a line. The comment character and everything else after it on the same line are ignored by the SQL client.

	; this comment spans an entire line
	select * from table ; this comment occurs after a valid statement

These allow you to comment your scripts and / or "comment out" a statement that you want to bypass.

## SQL Scripts

An SQL script is a text file with a `.sql` extension. The SQL client runs these scripts by reading and executing each line in turn until no more lines are present or an error is encountered.

### login.sql

This script, if present, will be run prior to the SQL client accepting commands. Typically it contains a number of set commands.

#### Sample Client Script

A typical client login.sql script, connecting to a RebDB server, would contain the following at a minimum.

	set address  tcp://127.0.0.1:1000
	set username Bob

#### Sample Server Script

A typical server login.sql script would contain the following at a minimum.

	set address  tcp://:1000
	set spool    server.log

## Reserved Words

The following words are reserved and may not be used as table or column names:

- avg
- by
- count
- desc
- distinct
- explain
- from
- group
- header
- having
- into
- joins
- max
- min
- on
- order
- replaces
- rowid
- set
- std
- sum
- table
- to
- values
- where
- with
