## MySQL一些常用的时间函数

### 一、获得当前时间

获取当前时间格式串：

```mysql
#获取当前时间戳
current_timestamp() yyyy-mm-dd hh:ii:ss 
now() yyyy-mm-dd hh:ii:ss     ##在执行开始时值就得到时间值
sysdate() yyyy-mm-dd hh:ii:ss ##在函数执行时动态得到时间值
curdate() yyyy-mm-dd
current_date()
curtime() hh:ii:ss
current_time()

mysql> select current_timestamp(), now(), curdate(), current_date(), curtime(), current_time();
+---------------------+---------------------+------------+----------------+-----------+----------------+
| current_timestamp() | now()               | curdate()  | current_date() | curtime() | current_time() |
+---------------------+---------------------+------------+----------------+-----------+----------------+
| 2017-09-18 07:09:34 | 2017-09-18 07:09:34 | 2017-09-18 | 2017-09-18     | 07:09:34  | 07:09:34       |
+---------------------+---------------------+------------+----------------+-----------+----------------+

mysql> select now(), sleep(3), sysdate();
+---------------------+----------+---------------------+
| now()               | sleep(3) | sysdate()           |
+---------------------+----------+---------------------+
| 2017-09-18 07:17:20 |        0 | 2017-09-18 07:17:23 |
+---------------------+----------+---------------------+
```

### 二、日期时间转换函数

日期转换为字符串函数，日期格式详见：[mysql日期格式](2.1mysql日期格式.md)

```mysql
## 日期时间转化为字符串
date_format(date,format)
time_format(time,format)

##字符串转化为日期
str_to_date(str, format)


select date_format(now(), '%Y%m%d%H%i%s'); ## 20170911165900
select date_format('2017-09-11 10:10:10', '%Y%m%d%H%i%s'); ## 20170911101010

select str_to_date('08/09/2008', '%m/%d/%Y'); ## 2008-08-09
select str_to_date('08:09:30', '%h:%i:%s'); ## 08:09:30
select str_to_date('08.09.2008 08:09:30', '%m.%d.%Y %h:%i:%s'); ## 2008-08-09 08:09:30
```


Unix时间戳和日期转换函数

```mysql
unix_timestamp() #unix时间戳1970-01-01以来的秒数
unix_timestamp('yyyy-mm-dd hh:ii:ss') #同时还可以将某一时间格式串的秒数转化出来
from_unixtime(unix_timestamp),
from_unixtime(unix_timestamp,format)

select unix_timestamp(); ## 1218290027
select unix_timestamp('2008-08-08'); ## 1218124800
select unix_timestamp('2008-08-08 12:30:00'); ## 1218169800

select from_unixtime(1218290027); ## '2008-08-09 21:53:47'
select from_unixtime(1218169800, '%Y %D %M %h:%i:%s %x'); ## '2008 8th August 12:30:00 2008'
```

提取date各个字段

```mysql
#提取date各个字段
date('yyyy-mm-dd hh:ii:ss') yyyy-mm-dd
year('yyyy-mm-dd hh:ii:ss') yyyy
month('yyyy-mm-dd hh:ii:ss') mm
day('yyyy-mm-dd hh:ii:ss') dd

#提取time各个字段
time('yyyy-mm-dd hh:ii:ss') hh:ii:ss
hour('yyyy-mm-dd hh:ii:ss') hh
minute('yyyy-mm-dd hh:ii:ss') ii
second('yyyy-mm-dd hh:ii:ss') ss
```

extract()从时间格式串中提取时间

```mysql

#提取天
select extract(DAY from now());
#提取天 小时 分钟 秒 
select extract(DAY_SECOND from now());
```

拼凑日期、时间函数

```mysql
makdedate(year, dayofyear)
maketime(hour, minute, second)

mysql> select makedate(2017, 31), makedate(2017, 32), maketime(12, 15, 30);
+--------------------+--------------------+----------------------+
| makedate(2017, 31) | makedate(2017, 32) | maketime(12, 15, 30) |
+--------------------+--------------------+----------------------+
| 2017-01-31         | 2017-02-01         | 12:15:30             |
+--------------------+--------------------+----------------------+
```

日期转换成天、秒数转换函数

```mysql
to_days(date)
from_days(days)

time_to_sec(time)
sec_to_time(seconds)

mysql> select to_days('0000-00-00'), to_days('2017-09-18'), from_days(736955);
+-----------------------+-----------------------+-------------------+
| to_days('0000-00-00') | to_days('2017-09-18') | from_days(736955) |
+-----------------------+-----------------------+-------------------+
|                  NULL |                736955 | 2017-09-18        |
+-----------------------+-----------------------+-------------------+

mysql> select time_to_sec('01:00:05'), sec_to_time(3605);
+-------------------------+-------------------+
| time_to_sec('01:00:05') | sec_to_time(3605) |
+-------------------------+-------------------+
|                    3605 | 01:00:05          |
+-------------------------+-------------------+
```

### 三、日期时间计算函数

```mysql
#+- 参照点 间隔度
#当前时间加5天
date_add(now(), interval 5 DAY)
#当前时间减1天
date_add(@dt, interval -1 day);
#当前时间减5天
date_sub(now(), interval 5 DAY)
#加5天2小时10分钟20秒
date_add(now(), interval "05 02:10:20" DAY_SECOND);
#加1年2个月
date_add(now(), interval "0001-02" YEAR_MONTH);
#当然也可以直接用运算符
select now() + interval 5 DAY, now() - interval 5 DAY
#换成时间戳的运算是
select unix_timestamp() + 5*24*60*60, unix_timestamp() - 5*24*60*60
```

datediff()返回两个时间点相差的天数

```mysql
#-5天 只会计算天数
select datediff(now(), now() + interval 5 day);
select datediff('2008-08-08', '2008-08-01');
```

timediff()返回两个时间点相差的时间

```mysql
select timediff('08:08:08', '00:00:00'); -- 08:08:08
select timediff('2008-08-08 08:08:08', '2008-08-08 00:00:00'); -- 08:08:08
```

时区（timezone）转换函数

```mysql
convert_tz(dt,from_tz,to_tz)

select convert_tz('2008-08-08 12:00:00', '+08:00', '+00:00'); -- 2008-08-08 04:00:00
select date_add('2008-08-08 12:00:00', interval -8 hour); -- 2008-08-08 04:00:00
select date_sub('2008-08-08 12:00:00', interval 8 hour); -- 2008-08-08 04:00:00
```
