---
layout: post
title: Deleting loads of MySQL table rows
---

Selectively deleting a large number of rows from a huge table, is not an easy task when we talk about relational databases.
There are many things you have to address:

- foreign key constraints
- indexes
- application performance
- replication performance

I think application performance is the most important, especially when we talk about online applications.

It's not uncommon to have to delete big loads of data, especially when there is a poor application design and implementation,
because of which many more rows than should are inserted into a table.

On the other hand, project specifications that include the phrase "for future use" regarding collecting data, are not rare.
Sometimes it means "we will store these, but never use them".

[#](#what-we-try-to-achieve){:.header-anchor}

## What we try to achieve

Our goal is to selectively clean a table from unnecessary data without affecting the application's performance.
We don't delete all rows, but **the most** of them.

[#](#start-by-testing){:.header-anchor}

## Start by testing

Testing things on production is a really bad idea. So, grab a backup of your production database and dump it into a non-production server.

The results of the tests have to be as much as close to realistic, so you should:

- Use the latest backup of the database.  
The size of the table you wish to clean matters on the duration.

- Use a server with identical resources  
Don't use a server with less or more resources as false performance metrics will be produced.

- Test what you actually want to do  
For example, you can't use a simple ```SELECT``` statement to measure the time a ```DELETE``` will take,
not even when they use the same criteria. It's not a reliable indication as ```SELECT``` in most cases doesn't lock tables and doesn't need
to write anything to disk, unless a temp table is needed.

- Document every step  
At the end of the day you should have a reproducible guide to run in the real world environment.

Although there are tools to do so, it's difficult to emulate the traffic you have on production even during low traffic hours.
If you want to try, check [jMeter](http://jmeter.apache.org/).

[#](#how-you-would-normally-do-it){:.header-anchor}

## How you would normally do it

You may say, "just execute a ```DELETE``` and boom, its done".

Although that's the most easy way to go, it will be an overkill for your servers.

If you try to delete millions of records from a huge table, executing ```mytop``` on the server will make you get stressed.
The most possible is that you will see queries stacked because the ```DELETE``` statement will eat-up most of the resources.

### Why it's not the solution

- Timed-out requests  
If the queries are issued from a web application, you must expect that the corresponding HTTP requests will get timed-out
if they exceed the 60 seconds.

- Too much I/O  
If you use MySQL + InnoDB (my case) you are about to have huge I/O as InnoDB logs all operations, in case a rollback is requested.
This will cost in performance, disk space and time.

- Freed space is not really freed<br />
```DELETE``` doesn't really give the space back to the operating system. To free it up ```OPTIMIZE TABLE``` has to be executed
which unfortunately will lock the table.

- Index "recalculation"  
The entries of the deleted rows must be removed from the indexes too. Any ```DELETE``` will trigger a "recalculation".
Each time a row is deleted **all** the indexes of the table have to be modified too. This will cost extra resources and time.

- Synchronization of replication  
In case you replicate your database, keep in mind that if the operation is slow in the master database, it will be slow in the slave databases too.
Slaves will not to be in sync with the master until the operation is finished on slaves.

[#](#you-feel-lucky){:.header-anchor}

## You feel lucky

If you really want to go with the simple ```DELETE```, which I don't recommend, it would be better if the fields in ```WHERE``` were part of an index.
This may save you some time from finding the rows you are about to delete.

It's really possible that you will not use an indexed field to choose what to delete. If that's your case, you are about
to have a really slow ```DELETE``` statement.

Creating indexes now will take a lot of time, extra disk space, I/O and you will have the table locked.

In case you have your table partitioned using the field you are about to use to find the rows, you are really lucky.

> Partitioning separates your rows based on a field. It will not affect your schema (ex. produce new tables)
but will only positively affect the queries using the partitioning key as they will search in smaller sets of data than the whole table or index.

If you wish to create partitions now, you will have to ```ALTER``` the table but you will have the table locked. Also the partition key
must be a primary key or part of it, so we add more ```ALTER```s. Also, if you want to do this now, introducing partitioning to a big
unpartitioned table will be slow.

If you still feel lucky and want to go with the ```DELETE``` way, do it in chunks.  
This will lead to shorter execution times that will progressively get shorter and shorter.

### My metrics

Using this way, deleting 10% of a table with 24 million rows, having 4 indexes and using InnoDB as a storage engine
took >30 minutes. Used it in production and many queries got stacked for ~40 seconds causing big delays to the interaction of the users with the application.

(The available server resources are irrelevant as the results of this method will be compared with the suggested solution).

[#](#the-suggested-solution){:.header-anchor}

## The suggested solution

A more performant option, in a case like mine where I wanted to delete 90% of the rows, is to move the data to be kept to a new table
and then switch the original table with the new.

I chose that way because its way quicker than the simple ```DELETE``` method. Also, in production, the application performance impact was between none and tiny,
only while moving the kept data to the new table.

The I/O was much less in my case where the largest part of the table had to be vanished.
Just compare performing an action on 22 million records (```DELETE```) against an action on 2 million records (```INSERT``` into the new table).  
Also, the old table was vanished with ```TRUNCATE``` instead of ```DELETE```.  
Because of this, no InnoDB logs were written and no ```OPTIMIZE TABLE``` was required to regain the space of the deleted rows.

Rolling-back is also easy. Just switch the tables back (available only before ```TRUNCATE```, of course!).

### My metrics

Copying 2 out of the 24 million rows, took ~12 minutes. So, to reverse that, it took ~12 minutes to selectively delete 22 million records!

You might think that copying the rows to a new table that has no indexes and create them later would be more performant.
In my case, it took the same time as I needed indexes on the new table and had to create them later.

```TRUNCATE``` and switching the tables were instant, less than 1 second each.

[#](#problems){:.header-anchor}

## Problems

### Downtime

The downtime was tiny in my case (less than 1 second) and was the time between switching the tables.
It's really-really small, but I have to note that.

### Latest inserted records may be missing from the new table

Whatever occurs between the time you start copying the data to the new table and the time you switch the tables, will be missing from the new table.
If a new record is inserted into the original table while copying, even if it matches the criteria of the rows to be kept, it won't be copied.

To overcome that, you can insert them after the whole process has finished. Select them by the primary key (if its an incremented ID)
or by a date field that stores the time the row was inserted.

### Latest updated row values are missing from the new table

It's true that this solution is not easy if the table you try to clean-up is updated frequently (in my case it isn't).
To overcome this, you have to know which rows were updated during the copy and update them later by creating ```UPDATE``` statements for each of them.
Track them in the application side by logging the IDs, or by a database column that holds the last update time, if you have any.

If you do the whole process during the least traffic time, the rows that need to be updated are just a few or none.

[#](#show-me-the-code){:.header-anchor}

## Show me the code

- Create the new table:

```
CREATE TABLE transactions_new (
...
) ENGINE=InnoDB AUTO_INCREMENT=XXX DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci
```

Replace "XXX" on the AUTO_INCREMENT option with the one that fits you. It has to be larger than the ```MAX(id)``` of your original table,
able to fit the missing rows, those inserted into the original table while copying. Else primary key violations will occur while copying the missing
data to the new table.

Also, don't use the same constraint names with the original table, you will not be allowed to do so.

- Copy the rows to the new table:

```
INSERT INTO transactions_new
SELECT * FROM transactions
WHERE status = 1;
```

- Get the ID of the latest row inserted into the new table, to find the rows inserted into the original table after you started copying:

```
SELECT MAX(id) FROM transactions_new;
-- write down the result
```

- Switch the tables:

```
RENAME TABLE transactions TO transactions_old;
RENAME TABLE transactions_new TO transactions;
```

- Get the missing data from the original table:

```
INSERT INTO transactions
SELECT * FROM transactions_old
WHERE status = 1
AND id > ?;
-- replace "?" with the before switching MAX(id) of the new table
```

- Vanish the original table if everything is fine:

```
TRUNCATE TABLE transactions_old;
DROP TABLE transactions_old;
```

[#](#the-donts){:.header-anchor}

## The don'ts

### Don't auto-create the new table

Don't just do this:

```
CREATE TABLE transactions_new
SELECT * FROM transactions
WHERE status = 1;
```

This will skip creating the primary keys, the foreign keys and the indexes. The behavior of your application will be unexpectable
after the tables are switched.

### Don't use web clients

Don't run this process from a web client because the queries will run for a long time and will get timed-out.
Prefer the MySQL CLI or any other non-web client.

[#](#other-useful-commands){:.header-anchor}

## Other useful commands

You won't normally need them, but who knows.

- Lock and unlock a table for write operations:

```
LOCK TABLES transactions WRITE;
-- do something during this time...
UNLOCK TABLES;
```

- Rollback (switch the tables back)

```
RENAME TABLE transactions TO transactions_new;
RENAME TABLE transactions_old TO transactions;
```

- Kill long running queries:

```
SELECT CONCAT('KILL ', id, ';')
FROM information_schema.processlist
WHERE time > 60
INTO OUTFILE '/tmp/long_runners';

SOURCE /tmp/long_runners;
```

[#](#conclusion){:.header-anchor}

## Conclusion

Regarding relational databases, the obvious way is not always the only or the most performant. Don't get surprised by crazy methods that do the job.

However, the line between the crazy working ideas and the bad ideas is thin.  
Don't get **that** crazy!
