---
title: mysql学习笔记
date: 2018-04-10 13:21:18
categories: MySQL
tags: [数据库, mysql]
copyright : true
---

### 基本了解：
mysql数据库为关系型数据库，个关系型数据库由一个或数个表格组成，表格中肯定有***键***（键(key): 表中用来识别某个特定的人物的方法, 键的值在当前列中具有唯一性。）
### 登录mysql（记得配置环境变量）
1. 管理员模式打开cmd
2. 启动服务net start mysql
3. 登录mysql -u root -p，回车之后根据提示输入密码
4. 建表等操作
5. 关闭服务net start mysql

**注意，可能会报错：**

```
C:\AppServ\MySQL> mysql -u root -p 
Enter password:  
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES
```
**解决方法如下：**

编辑mysql配置文件my.ini（在mysql的安装目录下，我的在D:\Program Files\MySQL\MySQL Server 5.0\my.ini），在[mysqld]这个条目下加入 skip-grant-tables 保存退出后重启mysql

1. 点击“开始”->“运行”(快捷键Win+R)。
2. 停止：输入 net stop mysql
3. 启动：输入 net start mysql  

这时候在cmd里面输入mysql -u root -p就可以不用密码登录了，出现 password：的时候直接回车可以进入，不会出现ERROR 1045 (28000)，但很多操作都会受限制，因为我们不能grant（没有权限）。

继续按下面的流程走：

1. 进入mysql数据库：

```
mysql> use mysql; 
Database changed
```

2. 给root用户设置新密码：  

```
mysql> update mysql.user set authentication_string=password("新密码") where user="root"; 
Query OK, 1 rows affected (0.01 sec) 
Rows matched: 1 Changed: 1 Warnings: 0
```

3. 刷新数据库

```
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```
 
4. 退出mysql：

```
mysql> quit; 
Bye
```
改好之后，再修改一下my.ini这个文件，把我们刚才加入的 "skip-grant-tables"这行删除，保存退出再重启mysql就可以了。

### 关于可视化工具HeidiSql的使用
1. 访问本机时候mysql时候，ip可以使本机ip或者默认的127.0.0.1,用户名为mysql之前设置的用户名，密码也是自己设置的，我的默认为root，******
2. 配置好后直接打开即可


### 创建库才能创建表

```
-- 创建一个名为 samp_db 的数据库，数据库字符编码指定为 gbk
create database samp_db character set gbk;
drop database samp_db; -- 删除 库名为samp_db的库
show databases;        -- 显示数据库列表。
use samp_db;     -- 选择创建的数据库samp_db
show tables;     -- 显示samp_db下面所有的表名字
describe 表名;    -- 显示数据表的结构
delete from 表名; -- 清空表中记录
```
### 创建表

```
CREATE TABLE `user_accounts` (
  `id`             int(100) unsigned NOT NULL AUTO_INCREMENT primary key,
  `password`       varchar(32)       NOT NULL DEFAULT '' COMMENT '用户密码',
  `reset_password` tinyint(32)       NOT NULL DEFAULT 0 COMMENT '用户类型：0－不需要重置密码；1-需要重置密码',
  `mobile`         varchar(20)       NOT NULL DEFAULT '' COMMENT '手机',
  `create_at`      timestamp(6)      NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
  `update_at`      timestamp(6)      NOT NULL DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
  -- 创建唯一索引，不允许重复
  UNIQUE INDEX idx_user_mobile(`mobile`)
)
ENGINE=InnoDB DEFAULT CHARSET=utf8
COMMENT='用户表信息';
```
**数据类型的属性解释：**

```
NULL：数据列可包含NULL值；

NOT NULL：数据列不允许包含NULL值；

DEFAULT：默认值；

PRIMARY：KEY 主键；

AUTO_INCREMENT：自动递增，适用于整数类型；

UNSIGNED：是指数值类型只能为正数；

CHARACTER SET name：指定一个字符集；

COMMENT：对表或者字段说明；
```
### 增删改查
##### select

```
SELECT 语句用于从表中选取数据。 
语法：SELECT 列名称 FROM 表名称 
语法：SELECT * FROM 表名称
```
##### update

```
Update 语句用于修改表中的数据。 
语法：UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值
```
##### insert

```
INSERT INTO 语句用于向表格中插入新的行。 
语法：INSERT INTO 表名称 VALUES (值1, 值2,....) 
语法：INSERT INTO 表名称 (列1, 列2,...) VALUES (值1, 值2,....)
```
##### delete

```
DELETE 语句用于删除表中的行。 
语法：DELETE FROM 表名称 WHERE 列名称 = 值
```
参考： [https://segmentfault.com/a/1190000006876419](https://segmentfault.com/a/1190000006876419)