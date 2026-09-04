# DCL——权限控制

MySQL 定义了多种权限，DCL 可以用于查询、授予和撤销用户权限。

## 1. 常用权限

| 权限 | 说明 |
|---|---|
| `ALL`、`ALL PRIVILEGES` | 指定范围内的所有权限 |
| `SELECT` | 查询数据 |
| `INSERT` | 插入数据 |
| `UPDATE` | 修改数据 |
| `DELETE` | 删除数据 |
| `ALTER` | 修改表结构 |
| `DROP` | 删除数据库、表或视图 |
| `CREATE` | 创建数据库或表 |

## 2. 权限作用范围

| 写法 | 作用范围 |
|---|---|
| `*.*` | 所有数据库中的所有对象 |
| `数据库名.*` | 指定数据库中的所有对象 |
| `数据库名.表名` | 指定数据库中的一张表 |

## 3. 查询用户权限

语法：

```sql
SHOW GRANTS FOR '用户名'@'主机名';
```

示例：

```sql
SHOW GRANTS FOR 'test_user'@'localhost';
```

## 4. 授予权限

语法：

```sql
GRANT 权限列表
ON 数据库名.对象名
TO '用户名'@'主机名';
```

授予用户查询和插入指定表的权限：

```sql
GRANT SELECT, INSERT
ON study_db.emp
TO 'test_user'@'localhost';
```

授予用户指定数据库中的所有权限：

```sql
GRANT ALL PRIVILEGES
ON study_db.*
TO 'test_user'@'localhost';
```

## 5. 撤销权限

语法：

```sql
REVOKE 权限列表
ON 数据库名.对象名
FROM '用户名'@'主机名';
```

撤销用户修改指定表的权限：

```sql
REVOKE UPDATE
ON study_db.emp
FROM 'test_user'@'localhost';
```

撤销用户在指定数据库中的所有权限：

```sql
REVOKE ALL PRIVILEGES
ON study_db.*
FROM 'test_user'@'localhost';
```

## 6. 注意事项

- `ALL PRIVILEGES` 表示指定作用范围内的全部权限，具体范围由 `ON` 后的对象决定。
- 用户名和主机名必须与创建账户时保持一致。
- 多个权限之间使用英文逗号分隔，例如 `SELECT, INSERT, UPDATE`。
- 实际使用时应遵循最小权限原则，只授予用户完成工作所必需的权限。
- 示例中的 `study_db` 应替换为实际数据库名。
