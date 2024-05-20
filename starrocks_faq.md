## Starrocks FAQ

### 1 主键表 DELETE的时候不能用where 传参
1. DELETE FROM imedw.dws_bos_testdrive_report_di WHERE dt='${date}';

2. DELETE FROM imedw.dws_bos_testdrive_report_di WHERE dt=date_add('${date}',1);
   
比如这种写法 第二条就不能执行，where后面的不能用函数处理

### 2 分区表只能静态分区不能动态分区
#### 表达式分区
自 v3.0 起，StarRocks 支持表达式分区（原称自动创建分区）
```
PARTITION BY expression
...
[ PROPERTIES( 'partition_live_number' = 'xxx' ) ]

expression ::=
    { date_trunc ( <time_unit> , <partition_column> ) |
      time_slice ( <partition_column> , INTERVAL <N> <time_unit> [ , boundary ] ) }
```
#### 动态分区
StarRocks 3.2 开始可以开启动态分区，需要手动创建支持动态分区的表
```
CREATE TABLE site_access(
event_day DATE,
site_id INT DEFAULT '10',
city_code VARCHAR(100),
user_name VARCHAR(32) DEFAULT '',
pv BIGINT DEFAULT '0'
)
DUPLICATE KEY(event_day, site_id, city_code, user_name)
PARTITION BY RANGE(event_day)(
PARTITION p20200321 VALUES LESS THAN ("2020-03-22"),
PARTITION p20200322 VALUES LESS THAN ("2020-03-23"),
PARTITION p20200323 VALUES LESS THAN ("2020-03-24"),
PARTITION p20200324 VALUES LESS THAN ("2020-03-25")
)
DISTRIBUTED BY HASH(event_day, site_id)
PROPERTIES(
    "dynamic_partition.enable" = "true",
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-3",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.history_partition_num" = "0"
);
```
### 3 流写 csv 模式对特殊字符处理会报错
最好json load
### 4 字符串长度有限制
- 对于 StarRocks 2.1 之前的版本，长度范围为 1~65533 字节。
- 【公测中】自 StarRocks 2.1 版本开始，长度范围为 1~1048576 字节。1048578（行最大值）- 2（长度标识位，记录实际数据长度）= 1048576
