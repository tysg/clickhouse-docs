---
title: How to use the exchange command to switch tables
description: “How to use the exchange command to switch tables“
date: 2024-11-21
---

# How to use the exchange command to switch tables

### Question

How do I use the EXCHANGE command to switch table names?

### Answer

The `EXCHANGE` command is useful when you need to switch a current table with another table that is temporary where possibly Primary Keys or other settings were updated.
This happens atomically vs with the `RENAME` command.
It is also useful when you have Materialized Views triggering on a source table and are trying to avoid rebuilding the view.

Below is a simple example on how it works and how to test:
- Create sample database
```
create database db1;
```
- Create example table
```
create table db1.table1_exchange
(
 id Int32,
 string_field String
)
engine = MergeTree()
order by id;
```
- Insert sample row
```
insert into db1.table1_exchange
values
(1, 'a');
```
- Create example temporary table that will be exchanged
```
create table db1.table1_exchange_temp
(
 id Int32,
 string_field String
)
engine = MergeTree()
order by id;
```
- Insert sample row into the temporary table
```
insert into db1.table1_exchange_temp
values
(2, 'b');
```
- Run the `EXCHANGE` command to switch the tables
```
exchange tables db1.table1_exchange and db1.table1_exchange_temp;
```
- Test that the tables are now exchanged and show the rows are switched
```
clickhouse-cloud :) select * from db1.table1_exchange;

SELECT *
FROM db1.table1_exchange

Query id: 925a9a54-ce0d-406f-9943-16930f770a65

┌─id─┬─string_field─┐
│  2 │ b            │
└────┴──────────────┘

1 row in set. Elapsed: 0.002 sec. 
```
Reference link:  
https://clickhouse.com/docs/en/sql-reference/statements/exchange