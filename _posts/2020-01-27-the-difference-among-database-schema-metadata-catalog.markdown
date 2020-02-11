---
layout: post
title:  "The difference among database Schema、Metadata and Catalog"
date:   2020-01-27 12:21:17 -0500
categories: Database
---
There are various explains about `database Schema`、`Metadata` and `Catalog`. Let's take MySQL as an example to summarize their difference.

`Schema` is a kind of layout of a database and interchangeable with the database. It refers to a container which consists of tables、views、constraints、triggers、procedures etc.
In MySQL, your database can be created by the following approaches:

```
create database my_db;
drop database my_db;

or

create schema my_db;
drop schema my_db;
```

`Metadata` is the data about data. For example, typical metadata include:
    - title
    - categories
    - author
    - datetime
    

`Catalog` is not used in MySQL, which is like a grouping of schemas. MySQL uses `INFORMATION_SCHEMA` to represent catalog concept. It can be used to query tables or other information in any schema.

```
SELECT TABLE_NAME, TABLE_ROWS
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = 'my_db';
```

See the reference of [INFORMATION_SCHEMA](https://dev.mysql.com/doc/refman/8.0/en/information-schema.html).
