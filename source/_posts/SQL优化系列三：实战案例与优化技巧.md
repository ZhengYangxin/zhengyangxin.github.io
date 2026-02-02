---
title: SQL优化系列（三）：实战案例与优化技巧
copyright: true
date: 2020-05-10 10:30:00
tags: [SQL优化, MySQL, 性能优化, 数据库]
category: 数据库
---

### 前言

前两篇文章分别介绍了EXPLAIN的使用和索引优化原则。本文将通过实际案例，演示SQL优化的完整过程，并分享一些实用的优化技巧。

<!-- more -->

### 案例一：分页查询优化

#### 问题场景

```sql
-- 当offset很大时，性能急剧下降
SELECT * FROM orders ORDER BY id LIMIT 1000000, 10;
```

这条SQL需要扫描1000010行数据，然后丢弃前1000000行，效率极低。

#### 优化方案

**方案1：使用子查询优化**

```sql
SELECT * FROM orders
WHERE id >= (SELECT id FROM orders ORDER BY id LIMIT 1000000, 1)
LIMIT 10;
```

**方案2：使用延迟关联**

```sql
SELECT o.* FROM orders o
INNER JOIN (SELECT id FROM orders ORDER BY id LIMIT 1000000, 10) AS t
ON o.id = t.id;
```

**方案3：记录上次查询的最大ID（推荐）**

```sql
-- 假设上一页最后一条记录的id是1000000
SELECT * FROM orders WHERE id > 1000000 ORDER BY id LIMIT 10;
```

### 案例二：COUNT优化

#### 问题场景

```sql
-- 统计订单总数，全表扫描
SELECT COUNT(*) FROM orders;

-- 统计某状态的订单数
SELECT COUNT(*) FROM orders WHERE status = 1;
```

#### 优化方案

**方案1：使用缓存**

对于不需要实时精确的场景，可以使用Redis缓存计数，定期更新。

**方案2：使用汇总表**

```sql
-- 创建汇总表
CREATE TABLE order_statistics (
    status TINYINT PRIMARY KEY,
    count INT
);

-- 通过触发器或定时任务维护
```

**方案3：使用EXPLAIN估算**

```sql
-- 获取大致行数（不精确但很快）
EXPLAIN SELECT * FROM orders;
-- 查看rows字段
```

**方案4：COUNT的不同写法**

```sql
-- 以下几种写法性能基本相同
SELECT COUNT(*) FROM orders;
SELECT COUNT(1) FROM orders;
SELECT COUNT(id) FROM orders;  -- 如果id可能为NULL，结果可能不同
```

### 案例三：JOIN优化

#### 问题场景

```sql
-- 多表关联查询
SELECT o.*, u.name, p.product_name
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
LEFT JOIN products p ON o.product_id = p.id
WHERE o.created_at >= '2020-01-01'
ORDER BY o.created_at DESC
LIMIT 100;
```

#### 优化方案

**1. 确保关联字段有索引**

```sql
-- 检查并添加索引
ALTER TABLE orders ADD INDEX idx_user_id (user_id);
ALTER TABLE orders ADD INDEX idx_product_id (product_id);
ALTER TABLE orders ADD INDEX idx_created_at (created_at);
```

**2. 小表驱动大表**

MySQL优化器通常会自动选择，但可以使用STRAIGHT_JOIN强制指定驱动表顺序。

```sql
SELECT STRAIGHT_JOIN o.*, u.name
FROM users u
INNER JOIN orders o ON o.user_id = u.id
WHERE u.id = 123;
```

**3. 避免在关联字段上使用函数**

```sql
-- 错误示例，索引失效
SELECT * FROM orders o
JOIN users u ON CAST(o.user_id AS CHAR) = u.id;

-- 正确做法
SELECT * FROM orders o
JOIN users u ON o.user_id = u.id;
```

### 案例四：ORDER BY优化

#### 问题场景

```sql
-- Extra显示Using filesort
SELECT * FROM orders WHERE user_id = 123 ORDER BY created_at DESC;
```

#### 优化方案

**创建复合索引**

```sql
-- 让ORDER BY也能使用索引
ALTER TABLE orders ADD INDEX idx_user_created (user_id, created_at);
```

**注意事项**：
- 索引列的顺序要与ORDER BY一致
- 升序/降序要一致（MySQL 8.0支持降序索引）
- 多列排序时，所有列的排序方向要一致

### 案例五：GROUP BY优化

#### 问题场景

```sql
-- 统计每个用户的订单数
SELECT user_id, COUNT(*) as order_count
FROM orders
GROUP BY user_id;
```

#### 优化方案

**1. 添加索引**

```sql
ALTER TABLE orders ADD INDEX idx_user_id (user_id);
```

**2. 使用松散索引扫描**

当GROUP BY的列是索引的最左前缀时，MySQL可以使用松散索引扫描，效率更高。

**3. 避免排序**

```sql
-- 如果不需要排序，可以禁用
SELECT user_id, COUNT(*) as order_count
FROM orders
GROUP BY user_id
ORDER BY NULL;
```

### 实用优化技巧

#### 1. 批量操作

```sql
-- 避免逐条插入
INSERT INTO orders (user_id, amount) VALUES (1, 100);
INSERT INTO orders (user_id, amount) VALUES (2, 200);

-- 使用批量插入
INSERT INTO orders (user_id, amount) VALUES
(1, 100), (2, 200), (3, 300);
```

#### 2. 避免SELECT *

```sql
-- 不推荐
SELECT * FROM orders WHERE id = 1;

-- 推荐，只查询需要的列
SELECT id, user_id, amount FROM orders WHERE id = 1;
```

#### 3. 使用LIMIT限制结果集

```sql
-- 如果只需要判断是否存在
SELECT 1 FROM orders WHERE user_id = 123 LIMIT 1;
```

#### 4. 合理使用UNION ALL

```sql
-- UNION会去重，需要排序
SELECT * FROM orders_2019 WHERE user_id = 123
UNION
SELECT * FROM orders_2020 WHERE user_id = 123;

-- UNION ALL不去重，性能更好
SELECT * FROM orders_2019 WHERE user_id = 123
UNION ALL
SELECT * FROM orders_2020 WHERE user_id = 123;
```

#### 5. 使用EXISTS代替IN

```sql
-- 当子查询结果集较大时，EXISTS通常更快
-- IN写法
SELECT * FROM users WHERE id IN (SELECT user_id FROM orders);

-- EXISTS写法
SELECT * FROM users u WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);
```

### SQL优化流程总结

1. **发现问题**: 通过慢查询日志、监控工具发现慢SQL
2. **分析执行计划**: 使用EXPLAIN分析SQL执行过程
3. **定位瓶颈**: 找出全表扫描、文件排序等问题
4. **优化方案**: 添加索引、改写SQL、调整表结构
5. **验证效果**: 对比优化前后的执行计划和执行时间
6. **持续监控**: 上线后持续关注SQL性能

### 总结

本系列文章从SQL问题发现、EXPLAIN使用、索引优化原则到实战案例，系统地介绍了SQL优化的方法和技巧。核心要点：

1. **善用EXPLAIN**: 分析执行计划是优化的基础
2. **合理设计索引**: 遵循最左前缀原则，避免索引失效
3. **优化查询语句**: 避免全表扫描，减少不必要的数据访问
4. **持续监控**: SQL优化是一个持续的过程

希望这个系列能帮助大家在实际工作中写出高效的SQL语句，提升系统性能。

### 参考资料

- MySQL官方文档
- 《高性能MySQL》
