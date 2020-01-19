---
title: "PostgreSQL 序列操作"
date: 2018-11-07T19:22:42+08:00
publishdate: 2018-11-07
lastmod: 2020-01-19
draft: false
tags: ["postgresql"]
---
```sql
# 查看当前序列的值：
SELECT currval('user_aticle_aid_seq');

# 设置序列的初始值：
select setval('user_aticle_aid_seq',100);

# 递增序列的值：
select nextval('user_aticle_aid_seq');
```