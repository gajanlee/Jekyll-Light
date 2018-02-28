---
layout : life
title: influxdb——时间序列数据库
category : influxdb
duoshuo: true
date : 2017-11-30
---

# 高阶
### 归档
* retention policy
创建保留策略为2 week用来包存10分钟一次的平均值归档。
```bash
CREATE RETENTION POLICY "two_week" ON "house" DURATION 2w REPLICATION 1
```
HTTP
```
https://twinpines-9429794e.influxcloud.net:8086/query?u=xiaogu&p=123q456w&db=house_data&pretty=true&q=CREATE%20RETENTION%20POLICY%20%22test%22%20ON%20%22house_data%22%20DURATION%201d%20REPLICATION%201
```
* continuous query
1. 切换数据库
```bash
use house
```
2. 创建连续查询
* 可以按照tag区分, group by *是所有的tag
```bash
create continuous query "cq_5m" on "house" begin select mean("Voltage") as "mean_voltage" into "two_week"."house_two_week_tags_test" from "house" group by time(5m), * end
```
注：其中`cq_10m`为连续查询的名称，`house`为数据库名，into后的`two_week`为house数据库上的保留策略，`house_two_week`为新值的`表(measurement)`，`house`为数据来源表，group by为10分钟，十分钟归档一次。

### 用户系统
1. "CREATE USER" user_name "WITH PASSWORD" password [ "WITH ALL PRIVILEGES" ] .
