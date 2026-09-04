# DQL——基础查询

DQL 的英文全称是 **Data Query Language（数据查询语言）**，用于查询数据库表中的数据。

## 1. DQL 完整语法

```sql
SELECT 字段列表
FROM 表名列表
WHERE 条件列表
GROUP BY 分组字段列表
HAVING 分组后条件列表
ORDER BY 排序字段列表
LIMIT 分页参数;
```

各子句的作用：

- `SELECT`：指定需要查询的字段。
- `FROM`：指定数据来源表。
- `WHERE`：对查询前的数据进行条件筛选。
- `GROUP BY`：按照指定字段分组。
- `HAVING`：对分组后的结果进行条件筛选。
- `ORDER BY`：对查询结果排序。
- `LIMIT`：限制返回的数据条数，常用于分页。

> 书写 SQL 时，关键字通常使用大写，以提高可读性。

## 2. 查询多个字段

### 查询指定字段

```sql
SELECT 字段1, 字段2, 字段3, ...
FROM 表名;
```

示例：

```sql
SELECT name, age, gender
FROM employee;
```

### 查询全部字段

```sql
SELECT *
FROM 表名;
```

示例：

```sql
SELECT *
FROM employee;
```

> `*` 表示全部字段。实际开发中通常建议明确写出字段名，查询结果更清晰，也能减少不必要的数据读取。

## 3. 设置字段别名

使用 `AS` 可以为查询结果中的字段设置别名：

```sql
SELECT 字段1 AS 别名1, 字段2 AS 别名2, ...
FROM 表名;
```

示例：

```sql
SELECT name AS 姓名, age AS 年龄
FROM employee;
```

`AS` 关键字可以省略：

```sql
SELECT name 姓名, age 年龄
FROM employee;
```

注意：

- 别名只改变查询结果中显示的列名，不会修改表中的原字段名。
- 如果别名中包含空格，可以使用反引号包裹，例如：``name AS `员工姓名` ``。

## 4. 去除重复记录

使用 `DISTINCT` 去除查询结果中的重复记录：

```sql
SELECT DISTINCT 字段列表
FROM 表名;
```

示例：查询员工表中所有不重复的职位：

```sql
SELECT DISTINCT job
FROM employee;
```

同时查询多个字段时，只有这些字段的组合完全相同，才会被视为重复记录：

```sql
SELECT DISTINCT job, gender
FROM employee;
```
