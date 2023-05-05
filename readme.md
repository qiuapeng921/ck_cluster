# ck同步文档

### 刷新表同步数据
```
OPTIMIZE TABLE micro_scrm.external_relation_day_local;
```

#### 查询集群信息
```
SELECT * from `system`.clusters c;
```

#### 创建库
```
CREATE database micro_scrm on cluster cluster_3s_1r;
```

#### 创建本地表
```
CREATE TABLE micro_scrm.external_relation_day_local on cluster cluster_3s_1r
(
    `corp_id` String,
    `external_user_id` String,
    `user_id` String,
    `add_time` Int64,
    `add_channel` Int64,
    `del_time` SimpleAggregateFunction(max, Int64)
) ENGINE = ReplicatedAggregatingMergeTree('/clickhouse/tables/micro_scrm.external_relation_day/{shard}', '{replica}')
PARTITION BY (corp_id, toYYYYMM(toDate(add_time))) ORDER BY (corp_id,external_user_id,user_id,add_time) SETTINGS index_granularity = 8192;
```

#### 创建分布式表
```
CREATE TABLE micro_scrm.external_relation_day on cluster cluster_3s_1r (
 `corp_id` String,
 `external_user_id` String,
 `user_id` String,
 `add_time` Int64,
 `add_channel` Int64,
 `del_time` SimpleAggregateFunction(max,Int64)
) ENGINE = Distributed('cluster_3s_1r', 'micro_scrm', 'external_relation_day_local', xxHash32(external_user_id))
```

#### 插入数据
```
INSERT INTO micro_scrm.external_relation_day_local (corp_id, external_user_id, user_id, add_time, add_channel, del_time) VALUES('wx123456', '外部联系人id-2', '15249279779', 0, 0, 0);
```



### 同步mysql表数据到ck表中
```
INSERT INTO micro_scrm.external_relation_day_local 
    SELECT 
    enterprise_id, 
    id,
    userid,
    created_at,
    `type`,
    deleted_at 
    FROM
mysql('localhost:3306', 'micro_scrm', 'members', 'root', '123456')

```

### 同步mysql库到ck中
```
CREATE DATABASE mysql ENGINE = MaterializedMySQL('localhost:3306', 'micro_scrm', 'root', '123456');
SHOW TABLES FROM mysql;
```