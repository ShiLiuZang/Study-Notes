# DQL——执行顺序

SQL 的编写顺序与数据库的实际执行顺序不同，需要注意区分。

## 1. 编写顺序

```sql
SELECT 字段列表
FROM 表名列表
WHERE 条件列表
GROUP BY 分组字段列表
HAVING 分组后条件列表
ORDER BY 排序字段列表
LIMIT 分页参数;
```

编写顺序为：

```text
SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY → LIMIT
```

## 2. 实际执行顺序

数据库处理查询时，逻辑执行顺序为：

```text
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

| 执行顺序 | 子句 | 作用 |
|---:|---|---|
| 1 | `FROM` | 确定数据来源 |
| 2 | `WHERE` | 对分组前的原始数据进行筛选 |
| 3 | `GROUP BY` | 对筛选后的数据进行分组 |
| 4 | `HAVING` | 对分组后的结果进行筛选 |
| 5 | `SELECT` | 选择需要返回的字段或计算结果 |
| 6 | `ORDER BY` | 对查询结果排序 |
| 7 | `LIMIT` | 限制最终返回的记录数量 |

## 3. 完整示例

统计成年员工较多的工作地址，按照人数从多到少排列，只返回前 3 条：

```sql
SELECT workaddress, COUNT(*) AS employee_count
FROM emp
WHERE age >= 18
GROUP BY workaddress
HAVING COUNT(*) >= 2
ORDER BY employee_count DESC
LIMIT 3;
```

这条 SQL 的执行过程：

1. `FROM emp`：读取员工表。
2. `WHERE age >= 18`：筛选成年员工。
3. `GROUP BY workaddress`：按照工作地址分组。
4. `HAVING COUNT(*) >= 2`：保留员工数不少于 2 的分组。
5. `SELECT ...`：返回工作地址和员工人数。
6. `ORDER BY employee_count DESC`：按照员工人数降序排列。
7. `LIMIT 3`：只返回前 3 条结果。

## 4. 易错点

- `WHERE` 在 `SELECT` 之前执行，因此通常不能在 `WHERE` 中使用 `SELECT` 定义的别名。
- `ORDER BY` 在 `SELECT` 之后执行，因此可以使用查询字段的别名进行排序。
- `WHERE` 用于分组前筛选数据，不能直接筛选聚合结果。
- `HAVING` 用于分组后筛选数据，通常与聚合函数配合使用。
- `LIMIT` 最后执行，因此它限制的是排序、筛选后的最终结果。
