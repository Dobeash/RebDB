# Quick Start

## Introduction

Follow these examples if you want to jump straight into things before reading the remainder of the documentation. Start the SQL client by running the `SQL.r` script, then follow the remainder of the examples prefixed with `SQL>` below.

Alternatively, load the RebDB scripts by starting a console session and typing `do %db.r`, then follow the code examples.

## Create a table with three columns and describe it

	SQL> create my-table [ID Date Note]
	SQL> describe my-table

	db-create my-table [ID Date Note]
	db-describe my-table

## Insert some rows

	SQL> insert my-table values [1 10-Jan-2004 "Note-1"]
	SQL> insert my-table values [next 9-Jan-2004 "Note-2"]
	SQL> insert my-table values [next 8-Jan-2004 "Note-3"]
	SQL> insert my-table values [next 7-Jan-2004 "Note-4"]
	SQL> insert my-table values [next 6-Jan-2004 "Note-5"]
	SQL> insert my-table values [next 5-Jan-2004 "Note-6"]
	SQL> insert my-table values [next 4-Jan-2004 "Note-7"]
	SQL> insert my-table values [next 3-Jan-2004 "Note-8"]
	SQL> insert my-table values [next 2-Jan-2004 "Note-9"]
	SQL> insert my-table values [next 1-Jan-2004 "Note-10"]
	SQL> commit my-table

	repeat i 10 [
		db-insert my-table reduce [i 11-Jan-2004 " i join "Note-" i]
	]

	db-commit my-table

## Select some rows

	SQL> select * from my-table
	SQL> select id from my-table
	SQL> select [date id] from my-table

	db-select * my-table
	db-select id my-table
	db-select [date id] my-table

## How about sorting them?

	SQL> select * from my-table order by date
	SQL> select * from my-table order by date desc

	db-select/order * my-table date
	db-select/order/desc * my-table id

## Now for a where clause

	SQL> select * from my-table where [id = 1]
	SQL> select * from my-table where [find [1 3 5] id]
	SQL> select * from my-table where [find note "-3"]
	SQL> select * from my-table where [date < (now/date)]

	db-select/where * my-table [id = 1]
	db-select/where * my-table [find [1 3 5] id]
	db-select/where * my-table [find note "-3"]
	db-select/where * my-table compose [date < (now/date)]

## What about multiple conditions?

	SQL> select * from my-table where [any [id < 3 date > 5-Jan-2004]]
	SQL> select * from my-table where [all [id < 3 date > 5-Jan-2004]]

	db-select/where * my-table [any [id < 3 date > 5-Jan-2004]]
	db-select/where * my-table [all [id < 3 date > 5-Jan-2004]]

## Specific rows?

	SQL> select * from my-table where [rowid = 1]
	SQL> select * from my-table where [rowid between 5 last]

	db-select/where * my-table [rowid = 1]
	db-select/where * my-table [rowid between 5 last]

## Update some rows

	SQL> update my-table set id to 0 where [id < 6]
	SQL> select * from my-table

	db-update/where my-table id 0 [id < 6]
	db-select * my-table

## Delete some rows

	SQL> delete from my-table where [id = 0]
	SQL> select * my-table

	db-delete/where my-table [id = 0]
	db-select * my-table

## Drop our table

	SQL> drop my-table

	db-drop my-table

These examples have just touched upon what RebDB can do, the remainder of the documentation covers these and other topics in greater detail.