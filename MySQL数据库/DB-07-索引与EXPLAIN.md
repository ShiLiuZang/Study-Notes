---
tags:
  - MySQL
  - 数据库
  - 索引
  - EXPLAIN
updated: "2026-08-15"
status: 已学习，待建索引与执行计划实操
---

# DB-07：索引与 EXPLAIN

> 本节目标：理解索引的用途和成本，掌握主键、唯一、普通与联合索引的基本区别，能够初步设计联合索引并阅读 `EXPLAIN`。建索引语法和执行计划对比留到真实数据实操继续巩固。

## 1. 索引解决什么问题

没有合适索引时，MySQL 可能需要检查大量记录；有合适索引时，可以利用有序结构快速定位满足条件的记录。

```text
没有索引 → 检查大量数据
有索引   → 先通过索引定位，再读取目标记录
```

InnoDB 的常规索引主要采用 B-tree 类结构，适合等值和范围条件，例如 `=`、`IN`、`BETWEEN`、`>` 和 `<`。

索引不是越多越好：

- 索引占用存储空间和缓存。
- 插入、更新和删除时需要维护索引。
- 过宽或重复索引会增加写入成本。
- 优化器不保证使用所有已创建的索引。

## 2. 常见索引类型

```text
PRIMARY KEY  主键索引，唯一且非空
UNIQUE       唯一索引，同时保证值不重复
INDEX        普通索引，只用于加速，不保证唯一
```

已有约束会产生索引：

```sql
PRIMARY KEY (id)
UNIQUE (user_id, name)
```

`CHECK` 负责校验值，不会创建查询索引。

查看索引：

```sql
SHOW INDEX FROM documents;
```

## 3. 聚簇索引、二级索引与回表

InnoDB 的表数据按照主键索引组织，主键索引也称为聚簇索引：

```text
主键值 → 完整行
```

二级索引的记录通常包含二级索引键和主键值：

```text
email → user.id
```

查询完整用户：

```sql
SELECT *
FROM users
WHERE email = 'alice@example.com';
```

大致过程：

```text
email 唯一索引 → 得到主键 id → 主键聚簇索引 → 完整用户行
```

最后根据主键读取完整行通常称为回表。主键值会出现在二级索引记录中，因此主键应尽量短小、稳定。

## 4. 联合索引与最左前缀

```sql
CREATE INDEX idx_documents_kb_deleted_created_id
ON documents (
    knowledge_base_id,
    deleted_at,
    created_at,
    id
);
```

联合索引按列顺序组织：

```text
knowledge_base_id → deleted_at → created_at → id
```

按最左前缀进行基础判断：

```text
knowledge_base_id                              可使用前缀
knowledge_base_id + deleted_at                 可使用前缀
knowledge_base_id + deleted_at + created_at    可使用前缀
deleted_at                                     跳过最左列
created_at                                     跳过前两列
```

索引中包含某字段，不代表单独查询该字段就一定能够高效使用整个联合索引。优化器可能采用其他访问策略，但设计时先掌握最左前缀规则。

`WHERE` 中条件的书写顺序通常不决定索引使用；索引定义的列顺序和查询语义才重要。

## 5. 等值列与范围列顺序

查询：

```sql
WHERE knowledge_base_id = 1
  AND status = 'ready'
  AND created_at BETWEEN '2026-08-01' AND '2026-08-31'
```

通常优先考虑：

```text
(knowledge_base_id, status, created_at)
```

两个等值列在前，范围列在后。到达第一个范围条件后，后续列通常难以继续用于缩小索引扫描范围。

## 6. 覆盖索引

假设索引为：

```text
(knowledge_base_id, status, created_at, id)
```

查询只需要索引中已有的列：

```sql
SELECT id, status, created_at
FROM documents
WHERE knowledge_base_id = 1
  AND status = 'ready';
```

MySQL 可能直接从索引完成查询，不再读取完整行。这称为覆盖索引。

如果查询还需要 `title`，通常需要回表。覆盖索引是“索引相对于某条查询的效果”，不是独立索引类型。不能为了覆盖而把 `raw_content MEDIUMTEXT` 等大字段随意塞入索引。

## 7. 常见索引使用障碍

对索引列执行函数：

```sql
WHERE DATE(created_at) = '2026-08-15'
```

通常不利于普通 B-tree 范围定位。改为：

```sql
WHERE created_at >= '2026-08-15 00:00:00'
  AND created_at <  '2026-08-16 00:00:00'
```

字符串模式：

```sql
LIKE 'MySQL%'     -- 前缀确定，可能使用 B-tree 范围
LIKE '%MySQL%'    -- 开头不确定，普通 B-tree 通常难以直接定位
```

还要注意：

- 跳过联合索引最左列。
- 索引列发生不合适的隐式类型转换。
- 单独索引区分度很低的状态列。
- 小表全表扫描可能比使用索引更便宜。

“存在索引”不等于“优化器一定使用索引”。

## 8. EXPLAIN

```sql
EXPLAIN FORMAT=TRADITIONAL
SELECT id, title
FROM documents
WHERE knowledge_base_id = 1
  AND status = 'ready';
```

重点字段：

| 字段 | 含义 |
| --- | --- |
| `type` | 表访问方式 |
| `possible_keys` | 可能使用的索引 |
| `key` | 最终选择的索引 |
| `rows` | 预计检查的行数 |
| `filtered` | 预计过滤后保留比例 |
| `Extra` | 覆盖索引、额外排序等信息 |

常见 `type`：

```text
const   主键或唯一索引定位单条
ref     非唯一索引等值查找
range   索引范围扫描
index   扫描整个索引
ALL     扫描整张表
```

`possible_keys` 只是候选，`key` 才是最终选中的索引。`key = NULL` 表示没有选择索引。`rows` 是估算，不是实际精确行数。`ALL` 表示全表扫描，不是“所有访问类型”。

## 9. EXPLAIN ANALYZE

```sql
EXPLAIN ANALYZE
SELECT id, title
FROM documents
WHERE knowledge_base_id = 1
  AND status = 'ready';
```

`EXPLAIN ANALYZE` 会真正执行查询，并显示实际耗时、实际行数和循环次数。若预计 `10000` 行、实际 `20` 行，估算明显不接近，应检查：

- 表统计信息是否过旧。
- 数据分布是否不均匀。
- 查询条件选择性。
- 索引列顺序。

更新统计信息：

```sql
ANALYZE TABLE documents;
```

`EXPLAIN ANALYZE` 会执行查询，不能在生产环境中随意用于昂贵查询。

## 10. CREATE INDEX 语法

固定结构：

```sql
CREATE INDEX 索引名称
ON 表名称 (索引列);
```

示例：

```sql
CREATE INDEX idx_documents_kb_status_created_id
ON documents (
    knowledge_base_id,
    status,
    created_at,
    id
);
```

索引名称常用 `idx_表名_列名`。语法只有一个 `ON`，表名是 `documents`。索引列优先根据 `WHERE`、连接和排序需求设计，不是把 `SELECT` 中所有返回列直接照搬进去。

删除普通索引：

```sql
DROP INDEX idx_documents_kb_status_created_id
ON documents;
```

本轮对 `CREATE INDEX` 语法仍不熟练，不在概念学习阶段反复卡住，统一放到固定数据实操继续补齐。

## 11. 项目候选索引

文档列表：

```sql
CREATE INDEX idx_documents_kb_deleted_created_id
ON documents (
    knowledge_base_id,
    deleted_at,
    created_at,
    id
);
```

按知识库和状态查询：

```sql
CREATE INDEX idx_documents_kb_status_created_id
ON documents (
    knowledge_base_id,
    status,
    created_at,
    id
);
```

最终是否保留两个相近索引，必须根据项目真实查询、数据分布、`EXPLAIN ANALYZE` 和写入成本决定，不能只凭名称决定。

## 12. 本轮易错点

- 认为联合索引 `(a, b, c)` 可以同样高效地支持 `b` 或 `c` 单独查询。
- 把联合索引最左前缀答案误选为包含跳过首列的查询。
- 不知道二级索引先找到主键，再由主键读取完整行。
- 把 `ALL` 解释成“所有类型”，正确含义是全表扫描。
- 只看到 `possible_keys` 就认为已经使用索引，忽略 `key`。
- `CREATE INDEX` 重复写两个 `ON`。
- 把表名写成 `document`，而实际表名是 `documents`。
- 把 `SELECT` 中的 `id、title、status、created_at` 直接当成索引设计依据，遗漏真正的过滤列 `knowledge_base_id`。

## 13. 当前完成状态

状态：`[已学习，待建索引与执行计划实操]`。

已经理解索引用途、最左前缀、等值与范围顺序、覆盖索引、常见障碍和 `EXPLAIN` 核心字段。仍需在 MySQL 实际环境中完成：

1. 独立写出 `CREATE INDEX` 与 `DROP INDEX`。
2. 生成至少万级、分布明确的测试数据并执行 `ANALYZE TABLE`。
3. 保存三份添加索引前后的执行计划。
4. 至少一份使用 `EXPLAIN ANALYZE` 比较预计行数和实际行数。
5. 说明为什么优化器可能选择全表扫描，以及为什么不能给所有字段建索引。

这些任务与 DB-01～DB-06 的真实运行验证一起放入 MySQL 路线项目，不因当前语法遗忘阻塞 DB-08。

## 14. 官方参考

- [MySQL 8.4：MySQL 如何使用索引](https://dev.mysql.com/doc/refman/8.4/en/mysql-indexes.html)
- [MySQL 8.4：列索引](https://dev.mysql.com/doc/refman/8.4/en/column-indexes.html)
- [MySQL 8.4：多列索引](https://dev.mysql.com/doc/refman/8.4/en/multiple-column-indexes.html)
- [MySQL 8.4：InnoDB 查询优化](https://dev.mysql.com/doc/refman/8.4/en/optimizing-innodb-queries.html)
- [MySQL 8.4：EXPLAIN](https://dev.mysql.com/doc/refman/8.4/en/explain.html)
- [MySQL 8.4：SHOW INDEX](https://dev.mysql.com/doc/refman/8.4/en/show-index.html)
