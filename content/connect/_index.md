---
Title: Redis Connect
linkTitle: Redis Connect
description:
weight: 71
alwaysopen: false
categories: ["Connect"]
aliases: /connect/
         /connect.md
---

Redis Connect helps import (or _ingest_&nbsp;) existing data into Redis Enterprise.   

Redis Connect extracts data from existing systems, such as relational database management systems (RDBMS).  The data is loaded into a Redis Connect instance and then transformed into a format suitable for a Redis database.

Redis Connect synchronizes a Redis database to the source data and performs live updates; that is, data in the Redis database updates when changes appear in the source data.

To learn more, see:

- [Redis Connect architecture and components]({{<relref "connect/architecture">}})

- [Install and set up Redis Connect]({{<relref "connect/install">}})

- [Upgrade Redis Connect components]({{<relref "connect/upgrade">}})


## Database support

For preview, Redis Connect uses Debezium 1.9 connectors to support the following data sources:

| Database   | Versions           |
| ---------- | ------------------ |
| Oracle     | 12c, 19c, 21c      |
| MySQL      | 5.7, 8.0.x         |
| Postgres   | 10, 11, 12, 13, 14 |
| SQL Server | 2017, 2019         |