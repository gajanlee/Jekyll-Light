---
layout : life
title: inch——influxdb测试工具
category : influx
duoshuo: true
date : 2017-11-08
---

series | memory
---- | ---
1000*2 | 150MB
1000*20 | 200MB
1000*200 | 900MB
1000*500 | 1.9GB
1000*1000 | 3.0GB

# IoT
```bash
inch -report-host http://localhost:8086 -v -p 30 -t "1000,2" -f 100
```
* Concurrency=`1`(200个节点发送数据)
* BatchSize=`5000`
* FieldNum=`100`
* Series=`1000*2`
* Points=`3000`
### 结果
* 硬盘：1.9GB
* 运行时长： 2358.7秒
* (2543.8 pt/sec | 254377.9 val/sec) errors: 0 | μ: 1.852564663s, 90%: 3.474647015s, 95%: 3.947272979s, 99%: 5.192380764s

# 金融测试
```bash
inch -report-host http://localhost:8086 -v -b 200000 -t "10000,20" -c 1 -p 720
```
* Concurrency=`1`
* BatchSize=`200K`
* FieldNum=`1`
* Series=`10K*20`
* Points=`720`，一分钟一个点，运行12小时的量。
### 结果
* 硬盘：1.2GB
* 运行时长：1201.3秒
* (119875.1 pt/sec | 119875.1 val/sec) errors: 0 | μ: 1.645184386s, 90%: 2.633023203s, 95%: 2.993008777s, 99%: 3.889885468s

# 模拟达能
```bash
inch -host https://twinpines-9429794e.influxcloud.net:8086 -password 123q456w -user xiaogu -v -b 1000 -c 20 -f 1 -t "1000,20" -p 100
```
* 结果
```
T=00000037 2000000 points written (53287.3 pt/sec | 53287.3 val/sec) errors: 0 | μ: 367.216902ms, 90%: 556.65386ms, 95%: 765.484122ms, 99%: 1.674279231s

Total time: 37.5 seconds
```
# 文件大小命令
```bash
sudo du -s -h /var/lib/influxdb/data/
```