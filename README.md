# RebDB v2.0.3

> Updated 13-Apr-2007

RebDB is a small but highly efficient RDBMS written entirely in Rebol/Base syntax, meaning that it will run on any platform that Rebol/Core runs on.

# Features

- **Works out of the box** - Just add `do %db.r` to your script and you are ready to go. No marathon installation/configuration/tuning sessions required!
- **Native Rebol storage** - Your data is stored and accessed as Rebol values which means that you have the full range of Rebol data-types at your fingertips!
- **Plays well** - The dozen or so functions that drive the database behave like any other Rebol function, accepting and returning Rebol values as you would expect.
- **Lean and mean** - The entire distribution weighs in at less than 20Kb of highly optimized and tuned Rebol/Base syntax. It can blaze through millions of rows a second!
- **No indexes** - RebDB loads an entire table into memory and ensures it is sorted when needed. This simple technique makes indexes redundant as RebDB is able to directly access rows in large chunks!
- **No locking** - RebDB serializes all requests. Not having to worry about locking means we can crunch through statements at a far greater rate.

## Documentation

> Updated 7-Sep-2008

- [Release Notes](CHANGES.md) - Describes changes made for each release.
- [Quick Start Guide](QUICK-START.md) - Runs through some simple examples to get you started quickly.
- [DB Guide](DB-GUIDE.md) - Describes the design and operation of RebDB.
- [SQL Guide](SQL-GUIDE.md) - Describes the SQL syntax used by RebDB.
