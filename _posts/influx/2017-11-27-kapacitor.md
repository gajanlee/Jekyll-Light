---
layout : life
title: kapacitor——可编辑脚本的自动报警程序
category : influx
duoshuo: true
date : 2017-11-27
---

# Kapacitor写TICKscript

```bash
kapacitor define house -tick house.tick -type stream -dbrp house.autogen `type也可以是batch`
kapacitor show house `查看任务的信息`
kapacitor enable house
```
```
# house.tick
stream
|from()
.measurement('house')
|eval(lambda: float("Global_active_power") * 1000.0 / 60.0 - float("Sub_metering_1") - float("Sub_metering_2") - float("Sub_metering_3"))
.as('consumer')
|influxDBOut()
.database('house')
.retentionPolicy('autogen')
.measurement('house')
```

## 注意
* influxdb 与 kapacitor 之间是订阅(subscription)关系，所以如果他们之间是跨机房的关系，需要在kapacitor的配置文件中指定hostname，在influxdb中用`show subscriptions`查看。
* kapacitor 有测试脚本功能，然后通过重放来检查脚本是否正确执行。

***

# 模仿Chronograf操作Kapacitor
* 脚本 list.tick
```javascript
var data = stream
    |from()
        .database('house_data')
        .retentionPolicy('autogen')
        .measurement('house')
        .where(lambda: ("metric" == 'ljz'))
    |eval(lambda: "deviation")
        .as('value')

var trigger = data
    |alert()
        .crit(lambda: "value" > 4)
        .message('Alert: power consumption')
        .id('Power_Consumption_Rule' + ':{{.Group}}')
        .idTag('alertID')
        .levelTag('level')
        .messageField('message')
        .durationField('duration')
        .details('The error of power consumption is greater than the threshold(6Wh)')
        .email('lee_jiazh@163.com', 'qingquan@xiaogu-tech.com')

trigger
    |influxDBOut()
        .create()
        .database('httpTest')
        .retentionPolicy('autogen')
        .measurement('httptest')
        .tag('alertName', 'test_http')
        .tag('triggerType', 'threshold')

trigger
    |httpPost('https://xg-grafana.herokuapp.com/api/test')
```
功能：向目标Post 一个Http请求，当trigger后向InfluxDB写数据。
1. InfluxDB 数据
```
time                alertID                    alertName duration     level    message                  metric triggerType value
----                -------                    --------- --------     -----    -------                  ------ ----------- -----
1511323486959072129 Power_Consumption_Rule:nil test_http 0            CRITICAL Alert: power consumption ljz    threshold   18.4
1511323498954461470 Power_Consumption_Rule:nil test_http 11995389341  CRITICAL Alert: power consumption ljz    threshold   27.96666666666667
```
2. Http Post的数据：
```json
{ series: 
     [ { name: 'house',
         tags: 
          { alertID: 'Power_Consumption_Rule:nil',
            level: 'CRITICAL',
            metric: 'ljz' },
         columns: [ 'time', 'duration', 'message', 'value' ],
         values: 
          [ [ '2017-11-22T05:14:26.541257857Z',
              1344412905118,
              'Alert: power consumption',
              20.733333333333334 ] ] } ],
    _id: '3oTa37s9Uva9swx5' },
```
## Tips
* 如果只想要状态改变时候触发，在alert()下加入`.stateChangesOnly()`

***
# Udf 实践
1. 写一个python2脚本，名为`ttest.py`,放在`/tmp/kapacitor_udf`中。
2. 从github上克隆下依赖的python-agent包。
```cmd
git clone https://github.com/influxdata/kapacitor.git /tmp/kapacitor_udf/kapacitor
```
3. 修改配置文件(`/etc/kapacitor/kapacitor.conf`),PYTHONPATH为以来的目录中的两个py文件。
```conf
[udf]
[udf.functions]
    [udf.functions.tTest]
        # Run python
        prog = "/usr/bin/python2"
        # Pass args to python
        # -u for unbuffered STDIN and STDOUT
        # and the path to the script
        args = ["-u", "/tmp/kapacitor_udf/ttest.py"]
        # If the python process is unresponsive for 10s kill it
        timeout = "10s"
        # Define env vars for the process, in this case the PYTHONPATH
        [udf.functions.tTest.env]
            PYTHONPATH = "/tmp/kapacitor_udf/kapacitor/udf/agent/py"
```
4. 安装agent依赖的环境和包
```
apt install python-pip
python2 -m pip install six scipy

# 安装protocol buffer
wget https://github.com/google/protobuf/releases/download/v3.5.0/protobuf-all-3.5.0.zip
unzip -o protobuf-python-3.5.0.zip
cd protobuf-3.5.0/
./configure
make 
make install
export LD_LIBRARY_PATH="/usr/local/lib"
# 看看有没有protoc命令，是否安装成功

# 安装python-protocol
git clone https://github.com/google/protobuf.git
unzip protobuf-master.zip
cd protobuf-master/python/
python2 setup.py build
python2 setup.py install
```
4. 重启kapacitor
5. 写一个TICKscript脚本
```js
// This TICKscript monitors the three temperatures for a 3d printing job,
// and triggers alerts if the temperatures start to experience abnormal behavior.

// Define our desired significance level.
var alpha = 0.001

// Select the temperatures measurements
var data = stream
    |from()
        .measurement('temperatures')
    |window()
        .period(5m)
        .every(5m)

data
    //Run our tTest UDF on the hotend temperature
    @tTest()
        // specify the hotend field
        .field('hotend')
        // Keep a 1h rolling window
        .size(3600)
        // pass in the alpha value
        .alpha(alpha)
    |alert()
        .id('hotend')
        .crit(lambda: "pvalue" < alpha)
        .log('/tmp/kapacitor_udf/hotend_failure.log')

// Do the same for the bed and air temperature.
data
    @tTest()
        .field('bed')
        .size(3600)
        .alpha(alpha)
    |alert()
        .id('bed')
        .crit(lambda: "pvalue" < alpha)
        .log('/tmp/kapacitor_udf/bed_failure.log')

data
    @tTest()
        .field('air')
        .size(3600)
        .alpha(alpha)
    |alert()
        .id('air')
        .crit(lambda: "pvalue" < alpha)
        .log('/tmp/kapacitor_udf/air_failure.log')
```
6. 写产生测试数据的脚本
```python
#!/usr/bin/python2

from numpy import random
from datetime import timedelta, datetime
import sys
import time
import requests


# Target temperatures in C
hotend_t = 220
bed_t = 90
air_t = 70

# Connection info
write_url = 'http://localhost:9092/write?db=printer&rp=autogen&precision=s'
measurement = 'temperatures'

def temp(target, sigma):
    """
    Pick a random temperature from a normal distribution
    centered on target temperature.
    """
    return random.normal(target, sigma)

def main():
    hotend_sigma = 0
    bed_sigma = 0
    air_sigma = 0
    hotend_offset = 0
    bed_offset = 0
    air_offset = 0

    # Define some anomalies by changing sigma at certain times
    # list of sigma values to start at a specified iteration
    hotend_anomalies =[
        (0, 0.5, 0), # normal sigma
        (3600, 3.0, -1.5), # at one hour the hotend goes bad
        (3900, 0.5, 0), # 5 minutes later recovers
    ]
    bed_anomalies =[
        (0, 1.0, 0), # normal sigma
        (28800, 5.0, 2.0), # at 8 hours the bed goes bad
        (29700, 1.0, 0), # 15 minutes later recovers
    ]
    air_anomalies = [
        (0, 3.0, 0), # normal sigma
        (10800, 5.0, 0), # at 3 hours air starts to fluctuate more
        (43200, 15.0, -5.0), # at 12 hours air goes really bad
        (45000, 5.0, 0), # 30 minutes later recovers
        (72000, 3.0, 0), # at 20 hours goes back to normal
    ]

    # Start from 2016-01-01 00:00:00 UTC
    # This makes it easy to reason about the data later
    now = datetime(2016, 1, 1)
    second = timedelta(seconds=1)
    epoch = datetime(1970,1,1)

    # 24 hours of temperatures once per second
    points = []
    for i in range(60*60*24+2):
        # update sigma values
        if len(hotend_anomalies) > 0 and i == hotend_anomalies[0][0]:
            hotend_sigma = hotend_anomalies[0][1]
            hotend_offset = hotend_anomalies[0][2]
            hotend_anomalies = hotend_anomalies[1:]

        if len(bed_anomalies) > 0 and i == bed_anomalies[0][0]:
            bed_sigma = bed_anomalies[0][1]
            bed_offset = bed_anomalies[0][2]
            bed_anomalies = bed_anomalies[1:]

        if len(air_anomalies) > 0 and i == air_anomalies[0][0]:
            air_sigma = air_anomalies[0][1]
            air_offset = air_anomalies[0][2]
            air_anomalies = air_anomalies[1:]

        # generate temps
        hotend = temp(hotend_t+hotend_offset, hotend_sigma)
        bed = temp(bed_t+bed_offset, bed_sigma)
        air = temp(air_t+air_offset, air_sigma)
        points.append("%s hotend=%f,bed=%f,air=%f %d" % (
            measurement,
            hotend,
            bed,
            air,
            (now - epoch).total_seconds(),
        ))
        now += second

    # Write data to Kapacitor
    r = requests.post(write_url, data='\n'.join(points))
    if r.status_code != 204:
        print >> sys.stderr, r.text
        return 1
    return 0

if __name__ == '__main__':
    exit(main())

```

7. 运行
```
kapacitor define print_temps -type stream -dbrp printer.autogen -tick print_temps.tick
kapacitor enable print_temps
cat /tmp/kapacitor_udf/{hotend,bed,air}_failure.log
```