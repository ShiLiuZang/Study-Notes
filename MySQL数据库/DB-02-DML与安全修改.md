---
tags:
  - MySQL
  - 数据库
  - DML
  - 事务
updated: "2026-08-14"
status: 已学习，待运行巩固
---

# DB-02：DML 与安全修改

> 本节目标：掌握数据的新增、修改和删除；避免无条件修改、越权修改和 SQL 注入；理解最小事务及回滚。

## 1. INSERT：插入数据

单条插入：

```sql
INSERT INTO users (email, password_hash)
VALUES ('alice@example.com', 'hash_alice');
```

批量插入时只写一次 `INSERT INTO`，多组值使用逗号分隔：

```sql
INSERT INTO users (email, password_hash)
VALUES
    ('bob@example.com', 'hash_bob'),
    ('carol@example.com', 'hash_carol');
```

`VALUE` 可以用于单条插入，但统一使用更常见的 `VALUES`。字符串使用单引号，每条完整 SQL 以分号结束。

如果数据库是空库，上述三名用户的自增 ID 通常是 `1、2、3`；不要在一般业务代码中依赖“第几行就是哪个 ID”。

## 2. 插入时会被约束检查

写入数据时，DB-01 定义的约束会真正发挥作用：

- 重复邮箱会违反 `UNIQUE`。
- 不存在的 `user_id` 或 `knowledge_base_id` 会违反外键。
- 缺少必填字段或写入 `NULL` 会违反 `NOT NULL`。
- 非法文档状态会违反 `CHECK`。
- 同一知识库的相同 `content_hash` 会违反联合唯一约束。
- 不同知识库可以保存相同 `content_hash`。

约束是数据库的最后防线，但应用仍应校验输入并把冲突转换为清晰的业务错误。

## 3. UPDATE：安全修改

基础语法：

```sql
UPDATE knowledge_bases
SET name = 'Python 后端资料',
    description = '保存 FastAPI 和 MySQL 笔记'
WHERE id = 1;
```

在多用户项目中，仅使用资源 ID 不足以证明当前用户拥有该资源。修改知识库时还要加入所有权条件：

```sql
UPDATE knowledge_bases
SET name = 'Python 后端资料',
    description = '保存 FastAPI 和 MySQL 笔记'
WHERE id = 1
  AND user_id = 1;
```

安全习惯：

1. 先使用完全相同的 `WHERE` 执行 `SELECT`，确认目标行。
2. 再执行 `UPDATE` 或 `DELETE`。
3. 检查受影响行数。
4. 业务脚本中避免无 `WHERE` 的修改和删除。

## 4. DELETE 与软删除

硬删除会真正移除记录：

```sql
DELETE FROM documents
WHERE id = 1
  AND knowledge_base_id = 1;
```

需要恢复、审计或后续补偿的文档通常采用软删除：

```sql
UPDATE documents
SET deleted_at = CURRENT_TIMESTAMP(6)
WHERE id = 1
  AND knowledge_base_id = 1
  AND deleted_at IS NULL;
```

查询活动文档：

```sql
SELECT *
FROM documents
WHERE knowledge_base_id = 1
  AND deleted_at IS NULL;
```

恢复已软删除文档：

```sql
UPDATE documents
SET deleted_at = NULL
WHERE id = 1
  AND knowledge_base_id = 1
  AND deleted_at IS NOT NULL;
```

`NULL` 必须使用 `IS NULL` 或 `IS NOT NULL` 判断，不能使用 `= NULL` 或 `!= NULL`。

## 5. 受影响行数

修改后要检查数据库报告的受影响行数：

- `1`：通常表示目标行成功修改。
- `0`：可能是目标不存在、所有权条件不匹配、已经被软删除，或者新值和旧值相同（具体表现也受客户端配置影响）。
- 大于 `1`：如果原本只打算修改一条记录，应立即检查 `WHERE` 条件。

同一条软删除语句执行第二次时，记录已经不满足 `deleted_at IS NULL`，所以影响 `0` 行。这个条件让操作具有基本的幂等性。

## 6. 参数化查询与 SQL 注入

程序确实可以拼接 SQL 字符串，但不能把外部输入直接拼入 SQL 结构。恶意输入可能改变查询条件，造成越权读取或修改。

危险示意：

```python
sql = f"SELECT * FROM users WHERE email = '{email}'"
```

参数化查询把 SQL 结构和值分开。以下是数据库驱动风格的示意代码，具体占位符取决于所用驱动：

```python
cursor.execute(
    "SELECT * FROM users WHERE email = %s",
    (email,),
)
```

参数化不是简单地“给字符串加引号”，而是让数据库驱动单独传递并正确处理参数值。表名、列名和排序方向不能直接作为普通值参数，应由程序从固定白名单中选择。

## 7. 最小事务：提交与回滚

事务把多条语句当作一个工作单元：

```sql
START TRANSACTION;

UPDATE documents
SET status = 'processing'
WHERE id = 1;

COMMIT;
```

- `START TRANSACTION`：开始事务。
- `COMMIT`：永久保存本次事务的修改，并结束事务。
- `ROLLBACK`：撤销当前尚未提交的事务修改，并结束事务。

失败回滚示例，假设初始状态为 `pending`：

```sql
START TRANSACTION;

UPDATE documents
SET status = 'processing'
WHERE id = 1;

UPDATE documents
SET status = 'unknown'
WHERE id = 1;

ROLLBACK;
```

`unknown` 违反状态 `CHECK`。失败的第二条语句不会替应用自动撤销前面已经成功的第一条语句，因此应用捕获错误后必须明确回滚。执行 `ROLLBACK` 后，状态恢复为 `pending`。

如果先执行 `COMMIT`，再执行 `ROLLBACK`，已经提交的数据不会恢复；例如提交 `processing` 后，最终状态仍是 `processing`。

自动提交、ACID、隔离级别、并发更新和锁统一放到 DB-06 学习。

## 8. 本轮易错点

- 批量插入的多组值之间漏写逗号。
- 连续写两条 `INSERT` 时漏写第一条语句末尾的分号。
- 把 `processing` 拼错成 `prcessing`。
- 使用 `deleted_at != NULL`，而不是 `deleted_at IS NOT NULL`。
- 恢复文档时误写 `deleted_at IS NULL`；恢复目标应该是已经删除的记录。
- 只按资源 ID 修改，没有加入用户或父资源范围条件。
- 认为某条语句失败后，事务中之前成功的语句一定自动全部回滚。
- 把“参数化查询”误解为 SQL 或程序不能拼接字符串；真正原则是外部输入不能直接改变 SQL 结构。

## 9. 当前完成状态

状态：`[已学习，待运行巩固]`。

已经完成概念和读代码练习：

- 单条与批量插入。
- 约束冲突判断。
- 所有权条件与安全修改。
- 软删除、恢复和受影响行数。
- 参数化查询。
- `COMMIT` 与 `ROLLBACK` 结果判断。

后续需要在 MySQL 临时 schema 中真正运行种子数据和失败回滚实验。运行成功后，再把状态更新为 `[已巩固]`；完整并发与隔离实验留到 DB-06。

## 10. 官方参考

- [MySQL 8.4：INSERT](https://dev.mysql.com/doc/refman/8.4/en/insert.html)
- [MySQL 8.4：UPDATE](https://dev.mysql.com/doc/refman/8.4/en/update.html)
- [MySQL 8.4：DELETE](https://dev.mysql.com/doc/refman/8.4/en/delete.html)
- [MySQL 8.4：Prepared Statements](https://dev.mysql.com/doc/refman/8.4/en/sql-prepared-statements.html)
- [MySQL 8.4：COMMIT 与 ROLLBACK](https://dev.mysql.com/doc/refman/8.4/en/commit.html)
