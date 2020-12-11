---
title: "腾讯云EMR-Presto"
subtitle: ""
layout: post
author: "LiuL"
header-style: text
tags:
  - EMR
  - Presto
---

cat /data/account.csv | clickhouse-client --query="INSERT INTO testdb.account FORMAT CSVWithNames" --password

clickhouse-client --query="insert into testdb.account FORMAT CSV" < /data/account.csv

