## mysql获得当前日期时间函数

### 一、获得当前时间

MySQL 获得当前日期+时间（date + time）函数：now()/sysdate()

sysdate() 日期时间函数跟 now() 类似，不同之处在于：now() 在执行开始时值就得到了， sysdate() 在函数执行时动态得到值。

```mysql
select now(), sleep(3), now();
+---------------------+----------+---------------------+
| now()               | sleep(3) | now()               |
+---------------------+----------+---------------------+
| 2017-09-11 16:53:18 |        0 | 2017-09-11 16:53:18 |
+---------------------+----------+---------------------+

select now(), sleep(3), sysdate();
+---------------------+----------+---------------------+
| now()               | sleep(3) | sysdate()           |
+---------------------+----------+---------------------+
| 2017-09-11 16:54:12 |        0 | 2017-09-11 16:54:15 |
+---------------------+----------+---------------------+
```

MySQL 获得当前时间戳函数：current_timestamp, current_timestamp()

```mysql
select current_timestamp, current_timestamp();
+---------------------+---------------------+
| current_timestamp   | current_timestamp() |
+---------------------+---------------------+
| 2017-09-11 16:55:19 | 2017-09-11 16:55:19 |
+---------------------+---------------------+
```

### 二、日期时间转换函数

#### Date/Time to Str(日期/时间转换为字符串)函数：date_format(date,format), time_format(time,format)

```mysql
select date_format(now(), '%Y%m%d%H%i%s'); ## 20170911165900
select date_format('2017-09-11 10:10:10', '%Y%m%d%H%i%s'); ## 20170911101010
```

#### Str to Date(字符串转换为日期)函数：str_to_date(str, format)

```mysql
select str_to_date('08/09/2008', '%m/%d/%Y'); -- 2008-08-09
select str_to_date('08/09/08' , '%m/%d/%y'); -- 2008-08-09
select str_to_date('08.09.2008', '%m.%d.%Y'); -- 2008-08-09
select str_to_date('08:09:30', '%h:%i:%s'); -- 08:09:30
select str_to_date('08.09.2008 08:09:30', '%m.%d.%Y %h:%i:%s'); -- 2008-08-09 08:09:30
```

#### (日期、天数)转换函数：to_days(date), from_days(days)

```mysql
select to_days('0000-00-00'); -- 0
select to_days('2008-08-08'); -- 733627
```

#### (时间、秒)转换函数：time_to_sec(time), sec_to_time(seconds)

```mysql
select time_to_sec('01:00:05'); -- 3605
select sec_to_time(3605); -- '01:00:05'
```

### 拼凑日期、时间函数：makdedate(year,dayofyear), maketime(hour,minute,second)

```mysql
select makedate(2001,31); -- '2001-01-31'
select makedate(2001,32); -- '2001-02-01'
select maketime(12,15,30); -- '12:15:30'
```

#### (Unix 时间戳、日期)转换函数

```mysql
unix_timestamp(),
unix_timestamp(date),
from_unixtime(unix_timestamp),
from_unixtime(unix_timestamp,format)
```

下面是示例：

```mysql
select unix_timestamp(); -- 1218290027
select unix_timestamp('2008-08-08'); -- 1218124800
select unix_timestamp('2008-08-08 12:30:00'); -- 1218169800

select from_unixtime(1218290027); -- '2008-08-09 21:53:47'
select from_unixtime(1218124800); -- '2008-08-08 00:00:00'
select from_unixtime(1218169800); -- '2008-08-08 12:30:00'

select from_unixtime(1218169800, '%Y %D %M %h:%i:%s %x'); -- '2008 8th August 12:30:00 2008'
```
