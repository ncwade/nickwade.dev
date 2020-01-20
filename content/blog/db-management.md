---
title: "Handling Database Migrations & Modifications"
date: 2020-01-20T07:00:00-05:00
draft: false
---

# Intro
Databases have been a black box to me for quiet some time. As my $EMPLOYER transitions to a more team owned stance on databases(as opposed to have them owned by a central team), I figured I should start to get more comfortable with them. Once question that has been on my mind for awhile is how manage modifications and migrations to your database.

# Current Method
First some background on our current method. We have a fantastic group of DBAs at current $EMPLOYER, they make all operations absolutely sing. They write SQL scripts for any operation that will modify database state and store these in git. Any stored procedures are written in a repeatable format, so they can be run as many times as needed with no impact. This has worked well, though there are a couple deficiencies. The repeatable scripts are great, though their dependencies are not clearly marked. The schema modification scripts are typically marked with some identifying information, but it is not immediately apparent when they were run or in what order. This makes rebuilding to the current state challenging. An additional side-effect is that it is hard to build a development instance of the database.

# Wanted Improvements
In ideal world, we could use a simple tool/CICD solution to manage these database tweaks. It would support the migrations/modifications, allow(better yet, require) rollback scripts, allow you to easily rebuild a development database to specified version, and load your stored procedures.

# Database Management Tools
There seem to be three broad catagories of tools; A) Application embedded tools such as DbUp or EntityFramework migrations, B) standalone CLI tools such as Flyway or dbmate, and C) database specific tools such as DACPAC. I dislike the idea of embedding your database schema/migrations in your application as it seems to mix concerns(my data layer should be separate from my business logic). I also dislike the idea of database specific tools such as DACPAC, as they tend to have the same learning curve as more general tools with the add benefit of vendor lock-in. This leaves us with standalone CI tools as the only real option.

# Preferred Direction
So digging into platform agnostic CLI tools there seems to be one perfect tool; Flyway. It supports versioned migrations, undo migrations, repeatable migrations(triggering off of the md5 of the migration) and even hooks for the process. However one of the best features(undo migrations) is locked behind a paywall of ~$1000 a year(for 10 schemas). Not a terrible price, but something to be considered. The second best option seems to be dbmate. This tool also supports migrations/undo migrations, but no hooks nor repeatable migrations. All features are free. Either tool seems viable, it mostly seems to come down to how much repeatable migrations are worth to you. Given time I'd really like to add repeatable migrations to dbmate, I feel that this would give me the best of both worlds.

# Links!
* [dbmate](https://github.com/amacneil/dbmate)
* [Flyway](https://flywaydb.org/)
* [DbUp](https://dbup.github.io/)
* [FluentMigrator](https://fluentmigrator.github.io/)
* [EntityFramework Migrations](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/?tabs=dotnet-core-cli)
* [DACPAC](https://docs.microsoft.com/en-us/sql/relational-databases/data-tier-applications/data-tier-applications?view=sql-server-ver15)

I should also re-iterate that I am not an expert, so take everything I have written with a grain of salt.