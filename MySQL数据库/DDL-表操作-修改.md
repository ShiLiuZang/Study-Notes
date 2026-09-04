## 添加字段

语法：

```
ALTER TABLE 表名
ADD 字段名 类型(长度) [COMMENT '注释'] [约束];
```


## 修改数据类型

```
ALTER TABLE 表名
MODIFY 字段名 新数据类型(长度);
```

示例：把 `emp` 表中的 `nickname` 字段修改为 `VARCHAR(50)`：

```
ALTER TABLE emp
MODIFY nickname VARCHAR(50);
```

## 修改字段名和字段类型

```
ALTER TABLE 表名
CHANGE 旧字段名 新字段名 类型(长度)
[COMMENT '注释'] [约束];
```

示例：将 `emp` 表中的 `nickname` 改名为 `username`，并修改为 `VARCHAR(30)`：

```
ALTER TABLE emp
CHANGE nickname username VARCHAR(30) COMMENT '用户名';
```
## 删除字段

语法：

```
ALTER TABLE 表名 DROP 字段名;
```

示例：删除 `emp` 表中的 `username` 字段：

```
ALTER TABLE emp DROP username;
```
## 删除表

```
DROP TABLE [IF EXISTS] 表名;
```

示例：

```
DROP TABLE IF EXISTS emp;
```

删除后，表结构和表内数据都会被删除。

## 清空并重新创建表

```
TRUNCATE TABLE 表名;
```