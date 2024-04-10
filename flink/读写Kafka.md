#Flink读写Kafka
##Kafka -> MySQL
### 注意事项
- [Flink](https://nightlies.apache.org/flink/flink-docs-master/docs/connectors/table/overview/)的kafka connector是支持同时读和写的
   - 读特有参数：scan.startup.mode等
   - 写特有参数：sink.partitioner等
- flink-connector-kafka-1.16.0.jar 和 flink-sql-connector-kafka-1.16.0.jar , 一个是用于代码方式，一种用于SQL方式，如果用sql-client提交任务，需要引入sql的jar
- 引入新的jar，需要重新启动集群和client
- flink 1.6.0 中引入kafka和jdbc需要[下载](https://mvnrepository.com/)下面的jar到lib目录：
   - flink-connector-jdbc-1.16.0.jar
   - flink-connector-kafka-1.16.0.jar
   - flink-sql-connector-kafka-1.16.0.jar
   - kafka-clients-3.5.0.jar
   - mysql-connector-j-8.0.33.jar

```
CREATE TABLE KafkaTable (
  `key` STRING,
  `value` STRING,
  `rowtime` TIMESTAMP(3) METADATA FROM 'timestamp',    -- use a metadata column to access Kafka's record timestamp
  `proctime` AS PROCTIME(),    -- use a computed column to define a proctime attribute
  WATERMARK FOR `rowtime` AS `rowtime` - INTERVAL '5' SECOND    -- use a WATERMARK statement to define a rowtime attribute
) WITH (
  'connector' = 'kafka',
  'properties.bootstrap.servers' = '127.0.0.1:9092',
  'topic' = 'test',
  'scan.startup.mode' = 'earliest-offset',
  'format' = 'json',
  'json.ignore-parse-errors' = 'true'
);


CREATE TABLE SinkTable (
    `key` STRING,
    `value` STRING
) WITH (
   'connector' = 'jdbc',
   'url' = 'jdbc:mysql://127.0.0.1:3306/flink?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&useSSL=false&zeroDateTimeBehavior=convertToNull&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true',
   'username' = 'root',
   'password' = '11111111',
   'table-name' = 'kafka_sink',
   'driver' = 'com.mysql.cj.jdbc.Driver'
);

insert into SinkTable select `key`,`value` from KafkaTable;
```
## 指定时间读
- 从指定的时间戳（毫秒）起（包括）scan.startup.timestamp-millis

```
CREATE TABLE KafkaTable (
  `key` STRING,
  `value` STRING,
  `rowtime` TIMESTAMP(3) METADATA FROM 'timestamp',    -- use a metadata column to access Kafka's record timestamp
  `proctime` AS PROCTIME(),    -- use a computed column to define a proctime attribute
  WATERMARK FOR `rowtime` AS `rowtime` - INTERVAL '5' SECOND    -- use a WATERMARK statement to define a rowtime attribute
) WITH (
  'connector' = 'kafka',
  'properties.bootstrap.servers' = '127.0.0.1:9092',
  'topic' = 'test',
  'scan.startup.mode' = 'timestamp',
  'scan.startup.timestamp-millis' = '1712744545833',
  'format' = 'json',
  'json.ignore-parse-errors' = 'true'
);
```