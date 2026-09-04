# DQL——条件查询

条件查询使用 `WHERE` 子句筛选满足指定条件的数据。

## 1. 语法

```sql
SELECT 字段列表
FROM 表名
WHERE 条件列表;
```

## 2. 比较运算符

| 比较运算符 | 功能 |
|---|---|
| `>` | 大于 |
| `>=` | 大于等于 |
| `<` | 小于 |
| `<=` | 小于等于 |
| `=` | 等于 |
| `<>` 或 `!=` | 不等于 |
| `BETWEEN ... AND ...` | 在某个范围之内，包含最小值和最大值 |
| `IN (...)` | 在 `IN` 后的列表中选择一个值 |
| `LIKE` | 模糊匹配 |
| `IS NULL` | 判断是否为 `NULL` |

`LIKE` 的常用占位符：

- `_`：匹配任意一个字符。
- `%`：匹配任意个字符，包括零个字符。

## 3. 逻辑运算符

| 逻辑运算符 | 功能 |
|---|---|
| `AND` 或 `&&` | 并且，多个条件同时成立 |
| `OR` 或 `\|\|` | 或者，多个条件中任意一个成立 |
| `NOT` 或 `!` | 非、不是 |

> 推荐优先使用 `AND`、`OR` 和 `NOT`，语义更加清晰。

## 4. 常用示例

查询年龄等于 88 岁的员工：

```sql
SELECT *
FROM emp
WHERE age = 88;
```

查询年龄在 15 到 20 岁之间的员工，包含 15 岁和 20 岁：

```sql
SELECT *
FROM emp
WHERE age BETWEEN 15 AND 20;
```

查询年龄小于 30 岁并且性别为女的员工：

```sql
SELECT *
FROM emp
WHERE age < 30 AND gender = '女';
```

查询工作地址为北京或上海的员工：

```sql
SELECT *
FROM emp
WHERE workaddress IN ('北京', '上海');
```

查询姓名为两个字的员工：

```sql
SELECT *
FROM emp
WHERE name LIKE '__';
```

查询身份证号为空的员工：

```sql
SELECT *
FROM emp
WHERE idcard IS NULL;
```

## 5. 注意事项

- `BETWEEN` 后应先写最小值，再写最大值，并且包含两个边界值。
- 判断空值不能使用 `= NULL`，应使用 `IS NULL` 或 `IS NOT NULL`。
- 同时使用 `AND` 和 `OR` 时，可以使用括号明确条件的执行顺序。
