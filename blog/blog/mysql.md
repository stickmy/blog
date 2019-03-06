---
title: mysql
date: 2017-01-05 11:31:21
type: post
tag: mysql
meta:
  - name: description
    content: mysql 使用记录
  - name: keywords
    content: mysql
---

## mysql创建数据库与创建用户以及授权
<!-- more -->
**针对`mysql5.6`以后的版本**

## 创建用户

1. 以管理员身份登录mysql
```
mysql -u root -p
```

2. 选择mysql database
```
use mysql;
```

3. 创建用户并设定密码
```
create user 'username'@'localhost' identified by 'urpassword';
```
此处`localhost`表示只能本地访问, 可以换为`%`, 表示匹配所有主机, 远程也能访问

4. 使操作生效
```
flush privileges;
```

## 创建数据库

1. 命令
```
create database [数据库名称] default character set utf8 collate utf8_general_ci;
```

`[]`为了说明使用, 不需要在命令中打出来

2. 参数说明:
- `character set X` --- 采用字符集`X`, 如果单独指定了这项参数, 而没有指定`collate`, 则采用`X`的默认校对规则.

- `collate Y` --- 采用校对规则`Y`.

- 如果两个都没指定, 则采用数据库默认的字符集和校对规则.

## 为用户赋予操作数据库的权限

```
grant all privileges on [数据库名称].* to 'username'@'localhost|%'
identified by 'urpassword';
```
- 也可以单独设置某项权限
```
grant insert, update, select, delete, create on [数据库名称].* to
'username'@'localhost|%' identified by 'urpassword';
```
- 赋予用户权限的同时, 允许其再授权
```
grant all privileges on [数据库名称].* to 'username'@'localhost|%'
with grant option identified by 'urpassword';
```
- 使操作生效
```
flush privileges;
```

## 删除和取消操作

1. 取消用户所有数据库的所有权限
```
revoke all on *.* from [username];
```
2. 删除用户
```
delete from mysql.user where user = 'username'
```
3. 删除数据库
```
drop database [databaseName|schemaName];
```