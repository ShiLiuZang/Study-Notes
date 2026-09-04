## DDL——数据库操作

### 查询

查询所有数据库：

```
SHOW DATABASES;
```

查询当前数据库：

```
SELECT DATABASE();
```

### 创建

```
CREATE DATABASE [IF NOT EXISTS] 数据库名
[DEFAULT CHARSET 字符集]
[COLLATE 排序规则];
```
### 删除

```
DROP DATABASE [IF EXISTS] 数据库名;
```
### 使用数据库

```
USE 数据库名;
```
方括号 `[]` 表示该部分是可选语法，实际输入时不需要写方括号。