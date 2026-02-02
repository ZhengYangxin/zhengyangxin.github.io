---
title: SQL优化系列（一）：如何发现SQL问题与EXPLAIN详解
copyright: true
date: 2020-05-02 10:30:00
tags: [SQL优化, MySQL, EXPLAIN, 数据库]
category: 数据库
---

### 前言

在后端开发中，SQL优化是一个非常重要的技能。一条慢SQL可能会导致整个系统的性能瓶颈，甚至引发线上故障。本文是SQL优化系列的第一篇，主要介绍如何发现SQL中的问题以及EXPLAIN命令的详细使用。

<!-- more -->

### 为什么要进行SQL优化

在实际开发中，我们经常会遇到以下场景：
- 页面加载缓慢，用户体验差
- 数据库CPU飙高，系统响应变慢
- 某些接口超时，影响业务正常运行

这些问题很多时候都是由慢SQL引起的。因此，掌握SQL优化技能对于后端工程师来说至关重要。

### 如何发现SQL问题

#### 1. 慢查询日志

MySQL提供了慢查询日志功能，可以记录执行时间超过指定阈值的SQL语句。

```sql
-- 查看慢查询日志是否开启
SHOW VARIABLES LIKE 'slow_query_log';

-- 开启慢查询日志
SET GLOBAL slow_query_log = 'ON';

-- 设置慢查询阈值（单位：秒）
SET GLOBAL long_query_time = 1;

-- 查看慢查询日志文件位置
SHOW VARIABLES LIKE 'slow_query_log_file';
```

#### 2. SHOW PROCESSLIST

通过`SHOW PROCESSLIST`命令可以查看当前正在执行的SQL语句，找出执行时间较长的查询。

```sql
SHOW PROCESSLIST;
-- 或者查看完整SQL
SHOW FULL PROCESSLIST;
```

重点关注以下字段：
- **Time**: 执行时间，单位秒
- **State**: 当前状态
- **Info**: 正在执行的SQL语句

#### 3. 性能监控工具

- **MySQL自带工具**: performance_schema
- **第三方工具**: pt-query-digest、MySQLTuner等

### EXPLAIN命令详解

EXPLAIN是分析SQL执行计划的核心工具，通过它可以了解MySQL如何执行一条SQL语句。

#### 基本用法

```sql
EXPLAIN SELECT * FROM users WHERE id = 1;
```

#### EXPLAIN输出字段详解

##### 1. id

查询的序列号，表示查询中执行SELECT子句或操作表的顺序。

- **id相同**: 执行顺序从上到下
- **id不同**: id值越大，优先级越高，越先执行
- **id为NULL**: 表示这是一个结果集，不需要使用它来进行查询

##### 2. select_type

查询的类型，主要用于区分普通查询、联合查询、子查询等。

| 类型 | 说明 |
|------|------|
| SIMPLE | 简单查询，不包含子查询或UNION |
| PRIMARY | 查询中包含子查询，最外层查询标记为PRIMARY |
| SUBQUERY | SELECT或WHERE中包含的子查询 |
| DERIVED | FROM子句中的子查询，MySQL会递归执行并将结果放入临时表 |
| UNION | UNION中的第二个或后面的SELECT语句 |
| UNION RESULT | UNION的结果 |

##### 3. table

显示这一行的数据是关于哪张表的。

##### 4. partitions

查询涉及到的分区。

##### 5. type（重要）

访问类型，表示MySQL如何查找表中的行。**性能从好到差依次为**：

```
system > const > eq_ref > ref > range > index > ALL
```

| 类型 | 说明 |
|------|------|
| system | 表只有一行记录，这是const类型的特例 |
| const | 通过索引一次就找到了，用于比较PRIMARY KEY或UNIQUE索引 |
| eq_ref | 唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配 |
| ref | 非唯一性索引扫描，返回匹配某个单独值的所有行 |
| range | 只检索给定范围的行，使用一个索引来选择行 |
| index | 全索引扫描，遍历索引树 |
| ALL | 全表扫描，性能最差 |

**优化目标**: 至少达到range级别，最好能达到ref级别。

##### 6. possible_keys

显示可能应用在这张表上的索引，但不一定被实际使用。

##### 7. key

实际使用的索引。如果为NULL，则没有使用索引。

##### 8. key_len

表示索引中使用的字节数，可以通过该值计算查询中使用的索引长度。在不损失精确性的情况下，长度越短越好。

##### 9. ref

显示索引的哪一列被使用了，如果可能的话，是一个常数。

##### 10. rows

MySQL认为必须检查的行数，用于估算查询需要扫描的行数。

##### 11. filtered

表示返回结果的行数占需读取行数的百分比，值越大越好。

##### 12. Extra（重要）

包含不适合在其他列中显示但十分重要的额外信息。

| 值 | 说明 |
|------|------|
| Using filesort | MySQL需要额外的排序操作，无法利用索引完成排序，**需要优化** |
| Using temporary | 使用了临时表保存中间结果，常见于ORDER BY和GROUP BY，**需要优化** |
| Using index | 表示使用了覆盖索引，避免了访问表的数据行，效率较高 |
| Using where | 使用了WHERE过滤条件 |
| Using index condition | 使用了索引下推优化 |

### EXPLAIN实战示例

假设有一张用户表：

```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50),
    email VARCHAR(100),
    age INT,
    created_at DATETIME,
    INDEX idx_name (name),
    INDEX idx_age (age)
);
```

#### 示例1：全表扫描

```sql
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
```

如果email字段没有索引，type会显示为ALL，表示全表扫描。

#### 示例2：使用索引

```sql
EXPLAIN SELECT * FROM users WHERE name = '张三';
```

由于name字段有索引，type会显示为ref。

#### 示例3：覆盖索引

```sql
EXPLAIN SELECT name FROM users WHERE name = '张三';
```

Extra会显示Using index，表示使用了覆盖索引。

### 总结

本文介绍了发现SQL问题的几种方法以及EXPLAIN命令的详细用法。掌握EXPLAIN是SQL优化的基础，通过分析执行计划，我们可以：

1. 了解SQL的执行过程
2. 发现潜在的性能问题
3. 为优化提供方向

下一篇文章将介绍索引优化的原则和常见的索引失效场景，敬请期待。

### 参考资料

- MySQL官方文档
- 《高性能MySQL》
