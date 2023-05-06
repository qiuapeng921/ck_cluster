# ck同步文档

### 刷新表同步数据
```
OPTIMIZE TABLE micro_scrm.member_departments_local;
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
CREATE TABLE micro_scrm.member_departments_local on cluster cluster_3s_1r
(
  `id` Int64,
  `member_id` Int64,
  `department_id` Int64,
  `enterprise_id` Int64,
  `created_at` Int32,
  `updated_at` Int32,
  `deleted_at` Int32
)
ENGINE = ReplicatedMergeTree('/clickhouse/tables/micro_scrm.member_departments/{shard}','{replica}')
PARTITION BY toYYYYMM(toDate(created_at))
ORDER BY id
SETTINGS index_granularity = 8192;
```

#### 创建分布式表
```
CREATE TABLE micro_scrm.member_departments on cluster cluster_3s_1r
(
  `id` Int64,
  `member_id` Int64,
  `department_id` Int64,
  `enterprise_id` Int64,
  `created_at` Int32,
  `updated_at` Int32,
  `deleted_at` Int32
)
ENGINE = Distributed('cluster_3s_1r','micro_scrm','member_departments_local',id);
```

#### 插入数据
```
INSERT INTO micro_scrm.member_departments_local (id, member_id, department_id, enterprise_id, created_at, updated_at,deleted_at) VALUES('wx123456', '外部联系人id-2', '15249279779', 0, 0, 0);
```



### 全量同步mysql表数据到ck表中
```
INSERT INTO micro_scrm.member_departments_local 
    SELECT 
    id,
    member_id, 
    department_id, 
    enterprise_id, 
    created_at, 
    updated_at,
    deleted_at 
    FROM
mysql('localhost:3306', 'micro_scrm', 'member_departments', 'root', '123456')
```

### 带条件同步mysql表数据到ck表中
```
INSERT INTO micro_scrm.member_departments_local 
    SELECT 
    id,
    member_id, 
    department_id, 
    enterprise_id, 
    created_at, 
    updated_at,
    deleted_at 
    FROM
mysql('140.143.67.38:3307', 'micro_scrm', 'member_departments', 'root', '19930921') WHERE id = 3442842692697600;
```

### 同步mysql库到ck中
```
CREATE DATABASE mysql ENGINE = MaterializedMySQL('localhost:3306', 'micro_scrm', 'root', '123456');
SHOW TABLES FROM mysql;
```