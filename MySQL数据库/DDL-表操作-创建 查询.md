## 查询当前数据库中的所有表

```
SHOW TABLES;
```

执行前需要先选择数据库：

```
USE itcast;
SHOW TABLES;
```

## 查询表结构

```
DESC 表名;
```
## 查询指定表的建表语句

```
SHOW CREATE TABLE 表名;
```

示例：

```
SHOW CREATE TABLE employee;
```



## SQL：DDL 表操作——创建

```
CREATE TABLE 表名 (
    字段1 字段1类型 [COMMENT '字段1注释'],
    字段2 字段2类型 [COMMENT '字段2注释'],
    字段3 字段3类型 [COMMENT '字段3注释'],
    ......
    字段n 字段n类型 [COMMENT '字段n注释']
) [COMMENT '表注释'];
```

说明：

- `CREATE TABLE`：创建数据表。
- 字段之间使用英文逗号 `,` 分隔。
- 最后一个字段后不加逗号。
- `COMMENT` 用来添加字段或表的注释。
- `[]` 表示可选内容，实际输入时不写方括号。