# REBOL Pseudo-Relational Database

RebDB is a small but highly efficient RDBMS written entirely in REBOL/Base syntax, meaning that it will run on any platform that REBOL/Core runs on.

# Features

- **Works out of the box** - Just add `do %db.r` to your script and you are ready to go. No marathon installation/configuration/tuning sessions required!
- **Native REBOL storage** - Your data is stored and accessed as REBOL values which means that you have the full range of REBOL data-types at your fingertips!
- **Plays well** - The dozen or so functions that drive the database behave like any other REBOL function, accepting and returning REBOL values as you would expect.
- **Lean and mean** - The entire distribution weighs in at less than 20Kb of highly optimized and tuned REBOL/Base syntax. It can blaze through millions of rows a second!
- **No indexes** - RebDB loads an entire table into memory and ensures it is sorted when needed. This simple technique makes indexes redundant as RebDB is able to directly access rows in large chunks!
- **No locking** - RebDB serializes all requests. Not having to worry about locking means we can crunch through statements at a far greater rate.

# Further Information

- **QUICK-START.md** runs through some simple examples to get you started quickly.
- **DB-GUIDE.md** describes the design and operation of RebDB.
- **SQL-GUIDE.md** describes the SQL syntax used by RebDB.