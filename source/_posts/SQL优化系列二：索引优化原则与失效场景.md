---
title: SQL优化系列（二）：索引优化原则与失效场景
copyright: true
date: 2020-05-7 10:30:00
tags: [SQL优化, MySQL, 索引, 数据库]
category: 数据库
---

### 前言

上一篇文章介绍了如何发现SQL问题以及EXPLAIN命令的使用。本文将深入讲解索引优化的原则以及常见的索引失效场景，帮助大家写出高效的SQL语句。

<!-- more -->

### 索引基础回顾

#### 什么是索引

索引是帮助MySQL高效获取数据的数据结构。可以简单理解为"排好序的快速查找数据结构"。

#### MySQL索引类型

- **B+Tree索引**: 最常用的索引类型，适用于全键值、键值范围和键前缀查找
- **Hash索引**: 只支持等值查询，不支持范围查询
- **全文索引**: 用于全文搜索
- **R-Tree索引**: 用于地理空间数据

### 索引优化原则

#### 1. 最左前缀原则

对于复合索引，MySQL会从最左边的列开始匹配。

```sql
-- 假设有复合索引 INDEX idx_abc (a, b, c)

-- 能使用索引
SELECT * FROM t WHERE a = 1;
SELECT * FROM t WHERE a = 1 AND b = 2;
SELECT * FROM t WHERE a = 1 AND b = 2 AND c = 3;

-- 不能使用索引
SELECT * FROM t WHERE b = 2;
SELECT * FROM t WHERE c = 3;
SELECT * FROM t WHERE b = 2 AND c = 3;
```

#### 2. 覆盖索引

如果查询的列都在索引中，MySQL可以直接从索引获取数据，无需回表查询。

```sql
-- 假设有索引 INDEX idx_name_age (name, age)

-- 使用覆盖索引，Extra显示Using index
SELECT name, age FROM users WHERE name = '张三';

-- 需要回表查询
SELECT * FROM users WHERE name = '张三';
```

**优化建议**: 尽量使用覆盖索引，减少SELECT *的使用。

#### 3. 索引选择性

索引选择性 = 不重复的索引值数量 / 表的总记录数

选择性越高，索引效率越高。例如：
- 主键的选择性为1，是最高的
- 性别字段只有男/女两个值，选择性很低，不适合建索引

#### 4. 前缀索引

对于长字符串，可以只索引前面部分字符，节省索引空间。

```sql
-- 对email字段的前10个字符建立索引
ALTER TABLE users ADD INDEX idx_email (email(10));
```

**注意**: 前缀索引无法用于ORDER BY和GROUP BY，也无法使用覆盖索引。

### 索引失效的常见场景

#### 1. 对索引列进行运算或函数操作

```sql
-- 索引失效
SELECT * FROM users WHERE YEAR(created_at) = 2020;
SELECT * FROM users WHERE age + 1 = 20;

-- 优化后，可以使用索引
SELECT * FROM users WHERE created_at >= '2020-01-01' AND created_at < '2021-01-01';
SELECT * FROM users WHERE age = 19;
```

#### 2. 使用不等于（!= 或 <>）

```sql
-- 索引可能失效
SELECT * FROM users WHERE status != 1;

-- 优化：如果status只有几个值，可以改写为
SELECT * FROM users WHERE status IN (0, 2, 3);
```

#### 3. IS NULL 和 IS NOT NULL

```sql
-- 可能导致索引失效
SELECT * FROM users WHERE name IS NULL;
SELECT * FROM users WHERE name IS NOT NULL;
```

**建议**: 设计表时尽量避免NULL值，使用默认值代替。

#### 4. LIKE以通配符开头

```sql
-- 索引失效
SELECT * FROM users WHERE name LIKE '%张';
SELECT * FROM users WHERE name LIKE '%张%';

-- 可以使用索引
SELECT * FROM users WHERE name LIKE '张%';
```

#### 5. 字符串不加引号

```sql
-- 假设phone字段是VARCHAR类型

-- 索引失效（发生隐式类型转换）
SELECT * FROM users WHERE phone = 13800138000;

-- 可以使用索引
SELECT * FROM users WHERE phone = '13800138000';
```

#### 6. OR条件连接

```sql
-- 如果or两边的列不都有索引，则索引失效
SELECT * FROM users WHERE name = '张三' OR email = 'test@example.com';

-- 优化：确保OR两边的列都有索引，或改写为UNION
SELECT * FROM users WHERE name = '张三'
UNION
SELECT * FROM users WHERE email = 'test@example.com';
```

#### 7. 范围查询后的列无法使用索引

```sql
-- 假设有复合索引 INDEX idx_abc (a, b, c)

-- b使用了范围查询，c无法使用索引
SELECT * FROM users WHERE a = 1 AND b > 10 AND c = 3;
```

### 索引设计建议

#### 1. 适合建索引的场景

- WHERE子句中经常使用的列
- ORDER BY子句中的列
- GROUP BY子句中的列
- 多表JOIN的关联列
- 选择性高的列

#### 2. 不适合建索引的场景

- 数据量很小的表
- 频繁更新的列
- 选择性很低的列（如性别、状态）
- 很少用于查询的列

#### 3. 复合索引设计原则

- 将选择性高的列放在前面
- 将等值查询的列放在范围查询的列前面
- 考虑覆盖索引，将常用查询列加入索引

### 实战案例

假设有订单表：

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    user_id BIGINT,
    status TINYINT,
    amount DECIMAL(10,2),
    created_at DATETIME
);
```

常见查询场景：

```sql
-- 查询某用户的订单
SELECT * FROM orders WHERE user_id = 123;

-- 查询某用户某状态的订单
SELECT * FROM orders WHERE user_id = 123 AND status = 1;

-- 查询某用户某时间段的订单
SELECT * FROM orders WHERE user_id = 123 AND created_at >= '2020-01-01';
```

**推荐索引设计**:

```sql
-- 复合索引，满足多种查询场景
ALTER TABLE orders ADD INDEX idx_user_status_created (user_id, status, created_at);
```

### 总结

本文介绍了索引优化的核心原则和常见的索引失效场景：

1. **最左前缀原则**: 复合索引从左到右匹配
2. **覆盖索引**: 减少回表查询
3. **索引失效场景**: 函数操作、隐式转换、LIKE通配符开头等
4. **索引设计**: 根据查询场景合理设计复合索引

下一篇文章将通过实际案例，演示SQL优化的完整过程，敬请期待。

### 参考资料

- MySQL官方文档
- 《高性能MySQL》
