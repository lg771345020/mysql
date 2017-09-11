##  mysql操作查询结果case when then else end用法举例

Case具有两种格式。简单Case函数和Case搜索函数。

```mysql 
##简单Case函数 
CASE sex 
    WHEN '1' THEN '男' 
    WHEN '2' THEN '女' 
    ELSE '其他' END 
    
##Case搜索函数 
CASE WHEN sex = '1' THEN '男' 
     WHEN sex = '2' THEN '女' 
    ELSE '其他' END 
```

还有一个需要注意的问题，Case函数只返回第一个符合条件的值，剩下的Case部分将会被自动忽略。 

```mysql
##比如说，下面这段SQL，你永远无法得到“第二类”这个结果 
CASE WHEN col_1 IN ( 'a', 'b') THEN '第一类' 
     WHEN col_1 IN ('a')       THEN '第二类' 
    ELSE'其他' END 
```

### 一，已知数据按照另外一种方式进行分组，分析。 

有如下数据： 

```mysql
DROP TABLE IF EXISTS test;
CREATE TABLE test(
	country VARCHAR(20) DEFAULT '' COMMENT '国家',
	sex TINYINT DEFAULT 1 COMMENT '性别：1男性，2女性',
	population int DEFAULT 0 COMMENT '人口'
);

INSERT INTO test(country, population) VALUES
('中国', 600),
('美国', 100),
('加拿大', 100),
('英国', 200),
('法国', 300),
('日本', 250),
('德国', 200),
('墨西哥', 50),
('印度', 250);
```

根据这个国家人口数据，统计亚洲和北美洲的人口数量。如果使用Case函数，SQL代码如下: 

```mysql
SELECT  
    CASE country 
            WHEN '中国'     THEN '亚洲' 
            WHEN '印度'     THEN '亚洲' 
            WHEN '日本'     THEN '亚洲' 
            WHEN '美国'     THEN '北美洲' 
            WHEN '加拿大'  THEN '北美洲' 
            WHEN '墨西哥'  THEN '北美洲' 
    ELSE '其他' END '洲',
    SUM(population) '人口' 
FROM test 
GROUP BY CASE country 
            WHEN '中国'     THEN '亚洲' 
            WHEN '印度'     THEN '亚洲' 
            WHEN '日本'     THEN '亚洲' 
            WHEN '美国'     THEN '北美洲' 
            WHEN '加拿大'  THEN '北美洲' 
            WHEN '墨西哥'  THEN '北美洲' 
            ELSE '其他' END; 

结果：                      
+--------+------+
| 洲     | 人口 |
+--------+------+
| 北美洲 |  250 |
| 其他   |  700 |
| 亚洲   | 1100 |
+--------+------+
```

根据人口数据划分等级，SQL如下：

```mysql
SELECT country 国家, 
CASE WHEN population>600 THEN '一' WHEN population>300 THEN '二' ELSE '三' END 人口等级 
FROM test;

结果： 
+--------+----------+
| 国家   | 人口等级 |
+--------+----------+
| 中国   | 二       |
| 美国   | 三       |
| 加拿大 | 三       |
| 英国   | 三       |
| 法国   | 三       |
| 日本   | 三       |
| 德国   | 三       |
| 墨西哥 | 三       |
| 印度   | 三       |
+--------+----------+
```

### 二，用一个SQL语句完成不同条件的分组
 
```mysql
DROP TABLE IF EXISTS device;
CREATE TABLE device(
	ip VARCHAR(32) DEFAULT '' COMMENT '设备ip' PRIMARY KEY,
	state TINYINT DEFAULT 0 COMMENT '设备状态：0离线，1正常，2告警',
	room_id LONG COMMENT '机房id'
);

INSERT INTO device(ip, state, room_id) VALUES
('1.1.1.1', 0, 1),
('1.1.1.2', 1, 1),
('1.1.1.3', 0, 1),
('1.1.1.4', 2, 2),
('1.1.1.5', 1, 1),
('1.1.1.6', 2, 3),
('1.1.1.7', 0, 3);
``` 

根据机房统计离线、正常、告警设备：

```mysql
SELECT room_id 
,COUNT(CASE WHEN state=0 THEN 'off' END) 'off'
,COUNT(CASE WHEN state=1 THEN 'on' END) 'on'
,COUNT(CASE WHEN state=2 THEN 'alarm' END) 'alarm'
,COUNT(1) total
FROM `device` GROUP BY room_id;

结果：
+---------+-----+----+-------+-------+
| room_id | off | on | alarm | total |
+---------+-----+----+-------+-------+
| 1       |   2 |  2 |     0 |     4 |
| 2       |   0 |  0 |     1 |     1 |
| 3       |   1 |  0 |     1 |     2 |
+---------+-----+----+-------+-------+
```

### 三，根据条件有选择的UPDATE

例，有如下更新条件 
①工资5000以上的职员，工资减少10% 
②工资在2000到4600之间的职员，工资增加15% 
很容易考虑的是选择执行两次UPDATE语句，如下所示 

```mysql
##条件1 
UPDATE Personnel SET salary = salary * 0.9 WHERE salary >= 5000; 
##条件2 
UPDATE Personnel SET salary = salary * 1.15 WHERE salary >= 2000 AND salary < 4600; 
```

但是事情没有想象得那么简单，假设有个人工资5000块。首先，按照条件1，工资减少10%，变成工资4500。接下来运行第二个SQL时候，因为这个人的工资是4500在2000到4600的范围之内， 需增加15%，最后这个人的工资结果是5175,不但没有减少，反而增加了。如果要是反过来执行，那么工资4600的人相反会变成减少工资。暂且不管这个规章是多么荒诞，如果想要一个SQL 语句实现这个功能的话，我们需要用到Case函数。代码如下: 

```mysql
UPDATE Personnel SET salary = 
CASE WHEN salary >= 5000 THEN salary * 0.9 
    WHEN salary >= 2000 AND salary < 4600 THEN salary * 1.15 
    ELSE salary END;
``` 

这里要注意一点，最后一行的ELSE salary是必需的，要是没有这行，不符合这两个条件的人的工资将会被写成NUll,那可就大事不妙了。在Case函数中Else部分的默认值是NULL，这点是需要注意的地方。

