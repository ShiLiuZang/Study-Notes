
## 1. 介绍

聚合函数将一列数据作为一个整体进行纵向计算，并返回一个计算结果。

## 2. 常见聚合函数

| 函数 | 功能 |
|---|---|
| `COUNT` | 统计数量 |
| `MAX` | 获取最大值 |
| `MIN` | 获取最小值 |
| `AVG` | 计算平均值 |
| `SUM` | 求和 |

## 3. 基本语法

```sql
SELECT 聚合函数(字段列表)
FROM 表名;
```

可以使用 `AS` 为计算结果设置别名：

```sql
SELECT 聚合函数(字段) AS 别名
FROM 表名;
```

## 4. 常用示例

统计员工总数：

```sql
SELECT COUNT(*) AS 员工总数
FROM emp;
```

统计填写了身份证号的员工数量：

```sql
SELECT COUNT(idcard) AS 已填写身份证号人数
FROM emp;
```

查询员工的最大年龄和最小年龄：

```sql
SELECT MAX(age) AS 最大年龄,
       MIN(age) AS 最小年龄
FROM emp;
```

查询员工的平均年龄：

```sql
SELECT AVG(age) AS 平均年龄
FROM emp;
```

计算所有员工年龄之和：

```sql
SELECT SUM(age) AS 年龄总和
FROM emp;
```

结合 `WHERE` 条件统计北京员工的平均年龄：

```sql
SELECT AVG(age) AS 北京员工平均年龄
FROM emp
WHERE workaddress = '北京';
```

## 5. 注意事项

- 除 `COUNT(*)` 外，聚合函数通常会忽略值为 `NULL` 的数据。
- `COUNT(*)` 统计表中的记录总数，包含字段值为 `NULL` 的记录。
- `COUNT(字段)` 只统计该字段不为 `NULL` 的记录数。
- `AVG` 和 `SUM` 通常用于数值类型字段。
