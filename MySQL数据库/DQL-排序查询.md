# DQL——排序查询

排序查询使用 `ORDER BY` 子句对查询结果进行升序或降序排列。

## 1. 语法

```sql
SELECT 字段列表
FROM 表名
ORDER BY 字段1 排序方式1, 字段2 排序方式2;
```

## 2. 排序方式

| 排序方式 | 功能 |
|---|---|
| `ASC` | 升序，默认值 |
| `DESC` | 降序 |

如果不指定排序方式，默认按照升序排列：

```sql
SELECT *
FROM emp
ORDER BY age;
```

以上语句等同于：

```sql
SELECT *
FROM emp
ORDER BY age ASC;
```

## 3. 常用示例

按照年龄从小到大排序：

```sql
SELECT *
FROM emp
ORDER BY age ASC;
```

按照入职日期从晚到早排序：

```sql
SELECT *
FROM emp
ORDER BY entrydate DESC;
```

先按照年龄升序排列；年龄相同时，再按照入职日期降序排列：

```sql
SELECT *
FROM emp
ORDER BY age ASC, entrydate DESC;
```

结合条件查询，按照年龄从大到小排列北京员工：

```sql
SELECT *
FROM emp
WHERE workaddress = '北京'
ORDER BY age DESC;
```

## 4. 注意事项

- 可以按照数字、字符串、日期等类型的字段排序。
- 多字段排序时，只有前一个字段的值相同，才会按照后一个字段继续排序。
- `ORDER BY` 通常写在 `WHERE`、`GROUP BY` 和 `HAVING` 之后，写在 `LIMIT` 之前。
