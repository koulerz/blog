---
title: "PostgreSQL 常用操作"
date: 2016-04-03T12:21:24+08:00
publishdate: 2016-04-03
lastmod: 2020-07-27
draft: false
tags: ["postgresql", "database", "command"]
---
## 增删改查 SQL 语句
```sql
# 插入数据
INSERT INTO user_tbl(name, signup_date) VALUES('张三', '2013-12-22');

# 查询记录
SELECT * FROM user_tbl;

# 更新数据
UPDATE user_tbl set name = '李四' WHERE name = '张三';

# 删除记录
DELETE FROM user_tbl WHERE name = '李四' ;
```

## 批量更新 SQL 语句
```sql
-- 批量更新一个字段
UPDATE mytable 
SET myfield = CASE id 
    WHEN 1 THEN 'value'
    WHEN 2 THEN 'value'
    WHEN 3 THEN 'value'
END
WHERE id IN (1,2,3)

-- 批量更新多个字段
UPDATE categories 
SET display_order = CASE id 
    WHEN 1 THEN 3 
    WHEN 2 THEN 4 
    WHEN 3 THEN 5 
END, 
title = CASE id 
    WHEN 1 THEN 'New Title 1'
    WHEN 2 THEN 'New Title 2'
    WHEN 3 THEN 'New Title 3'
END
WHERE id IN (1,2,3)
```

## 序列操作 SQL 语句
```sql
# 查看当前序列的值
SELECT currval('user_id_seq');

# 设置序列的初始值为 100
select setval('user_id_seq',100);

# 递增序列的值并返回
select nextval('user_id_seq');
```

## 查询 SQL 语句
```sql
# 转换时间戳为格式化时间函数 TO_TIMESTAMP()
SELECT TO_TIMESTAMP(created_at) FROM user;

# 转换时间戳为格式化时间函数 TO_TIMESTAMP() ，显示为中国时区
# 使用 'CST' 时区不一定显示为中国本地时间，建议使用 'Asia/Shanghai' 时区
SELECT TO_TIMESTAMP(created_at) AT TIME ZONE 'CST' FROM user;
SELECT TO_TIMESTAMP(created_at) AT TIME ZONE 'Asia/Shanghai' FROM user;

# 返回字段中的条件判断
SELECT name, CASE sex WHEN 1 THEN '男' WHEN 2 THEN '女' ELSE '保密' END AS sex, age FROM user;

# 拼接字符串
SELECT CONCAT('姓名:', name) AS name, age FROM user;

# 数值计算
SELECT id, amount/100 FROM order;

# 保留 2 位小数精度
SELECT id, round(100/3, 2) FROM order;

# 解析数组格式数据，获取数据时先将数组转为 JSON，再用编程语言函数将 JSON 转为可识别格式，例如 PHP 中的 json_decode()
SELECT array_to_json(ids) FROM user;
```

## Schema 操作 SQL 语句
```sql
# 创建新表
CREATE TABLE user_tbl(name VARCHAR(20), signup_date DATE);

# 添加字段
ALTER TABLE user_tbl ADD email VARCHAR(40);
ALTER TABLE user_tbl ADD COLUMN images jsonb DEFAULT '{}';

# 更改字段类型
ALTER TABLE user_tbl ALTER COLUMN signup_date SET NOT NULL;

# 更改字段类型长度
ALTER TABLE user_tbl ALTER COLUMN password TYPE varchar(32);

# 为字段添加索引
CREATE INDEX index_name ON user_tbl (name);

# 设置字段默认值（注意字符串使用单引号）
ALTER TABLE user_tbl ALTER COLUMN email SET DEFAULT 'example@example.com';

# 设置字段默认值为指定序列（通常用于主键字段）
ALTER TABLE user_tbl ALTER COLUMN id SET DEFAULT nextval('user_id_seq');

# 去除字段默认值
ALTER TABLE user_tbl ALTER email DROP DEFAULT;

# 去除字段不为空限制
ALTER TABLE user_tbl ALTER email DROP NOT NULL;

# 重命名字段
ALTER TABLE user_tbl RENAME COLUMN signup_date TO signup;

# 删除字段
ALTER TABLE user_tbl DROP COLUMN email;

# 表重命名
ALTER TABLE user_tbl RENAME TO backup_tbl;

# 删除表
DROP TABLE IF EXISTS backup_tbl;

# 删除库
DROP DATABASE IF EXISTS hello;
```

## 创建数据表例子
```sql
CREATE TABLE lm_user(
    id SERIAL PRIMARY KEY NOT NULL, -- 'ID'
    openid VARCHAR(20) DEFAULT NULL UNIQUE, -- 'OPENID'
    email VARCHAR(50) NOT NULL UNIQUE, -- 'email'
    mobilephone CHAR(15) DEFAULT NULL UNIQUE, -- '手机号码'
    password VARCHAR(60) NOT NULL, -- '登录密码'
    name VARCHAR(20) DEFAULT NULL, -- '姓名'
    num INT NOT NULL DEFAULT '0', -- ''
    status SMALLINT NOT NULL DEFAULT '1' -- '状态'
);
```

## 常用控制台命令
```
\password           设置密码。
\q                  退出。
\h                  查看SQL命令的解释，比如\h select。
\?                  查看psql命令列表。
\l                  列出所有数据库。
\c [database_name]  连接其他数据库。
\d                  列出当前数据库的所有表格。
\d [table_name]     列出某一张表格的结构。
\du                 列出所有用户。
\e                  打开文本编辑器。
\conninfo           列出当前数据库和连接的信息。
```

## 备份和恢复
```
pg_dump     备份
pg_dumpall  备份所有数据库
pg_restore  恢复
psql exampledb < exampledb.sql 导入数据
```

## 参考 & 扩展阅读
- [Sequence 和主键有什么区别](http://bbs.csdn.net/topics/340077504)
- [PostgreSQL 序列（SEQUENCE）](http://www.cnblogs.com/mchina/archive/2013/04/10/3012493.html)
