# DCL——管理用户

DCL 的英文全称是 **Data Control Language（数据控制语言）**，主要用于管理数据库用户及其访问权限。

MySQL 账户由用户名和主机名共同确定：

```text
'用户名'@'主机名'
```

- `'用户名'@'localhost'`：只允许从本机连接。
- `'用户名'@'%'`：允许从任意主机连接，实际使用时应结合权限和网络策略谨慎配置。

## 1. 查询用户

课件写法：

```sql
USE mysql;

SELECT *
FROM user;
```

也可以直接查询 `mysql.user`，并只返回常用字段：

```sql
SELECT User, Host, plugin
FROM mysql.user;
```

## 2. 创建用户

语法：

```sql
CREATE USER '用户名'@'主机名'
IDENTIFIED BY '密码';
```

示例：创建只能从本机登录的用户：

```sql
CREATE USER 'test_user'@'localhost'
IDENTIFIED BY 'StrongPassword_123!';
```

避免用户已存在时报错，可以使用 `IF NOT EXISTS`：

```sql
CREATE USER IF NOT EXISTS 'test_user'@'localhost'
IDENTIFIED BY 'StrongPassword_123!';
```

> 新创建的用户默认没有业务数据库的访问权限，需要再使用 `GRANT` 授予权限。

## 3. 修改用户密码

推荐写法：使用服务器的默认身份验证插件修改密码。

```sql
ALTER USER '用户名'@'主机名'
IDENTIFIED BY '新密码';
```

示例：

```sql
ALTER USER 'test_user'@'localhost'
IDENTIFIED BY 'NewStrongPassword_456!';
```

课件中的旧版写法：

```sql
ALTER USER '用户名'@'主机名'
IDENTIFIED WITH mysql_native_password BY '新密码';
```

### 版本说明

`mysql_native_password` 已从 MySQL 8.0.34 起弃用，在 MySQL 8.4 中默认禁用，并在 MySQL 9.0 中移除。新版 MySQL 应优先使用不指定插件的 `IDENTIFIED BY` 写法，让服务器采用默认身份验证插件。

## 4. 删除用户

语法：

```sql
DROP USER '用户名'@'主机名';
```

示例：

```sql
DROP USER 'test_user'@'localhost';
```

避免用户不存在时报错，可以使用 `IF EXISTS`：

```sql
DROP USER IF EXISTS 'test_user'@'localhost';
```

## 5. 注意事项

- 用户名和主机名共同组成账户，`'test_user'@'localhost'` 与 `'test_user'@'%'` 是两个不同账户。
- 创建、修改和删除用户需要当前登录账户具有相应的管理权限。
- 创建用户不会自动授予业务数据库权限，需要另行执行 `GRANT`。
- `DROP USER` 会删除该账户及其权限，但不会自动删除该用户创建的数据库对象。
- 不要在学习环境之外直接使用示例密码，应替换为符合安全策略的强密码。

