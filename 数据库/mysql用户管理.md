
#### 一、概要

mysql 是一个多用户数据库，具有功能强大的访问控制系统，可以为不同用户指定允许的权限。mysql 用户可以分为普通用户和 root 用户。root 用户是超级管理员，拥有所有的权限，包括创建用户、删除用户和修改用户的密码等管理权限。普通用户只拥有被授予的各种权限。


#### 二、权限表

> user 表

user 表是 mysql 中最重要的一个权限表，记录允许链接到服务器的账号信息，里面的 **权限** 是全局的。例如一个用户在 user 表中被赋予了  DELETE 权限，则该用户可以删除 mysql 服务器中所有数据库中的任何记录。

user 表的字段大致可分为 用户列、权限列、安全列和资源控制列。


> db 表 和 host 表

db 表和 host 表是 mysql 数据库中非常重要的权限表。db 表中存储了用户对某个数据库的操作权限，决定用户能够从哪个主机存取哪个数据库。host 表中存储了某个主机对数据库的操作的权限，配合 db 权限表对给定主机上的数据库操作权限做更细致的控制。这个权限表不受 GRANT 和 REVOKE 语句的影响。db 表比较常用，host 表一般很少使用。db 表和 host 表的结构相似，字段大致可以分成两类：用户列和权限列。


> tables_priv 表和 columns_priv 表

tables_priv 表用来对表设置操作权限，columns_priv 表用来对表的某一列设置权限。


> procs_priv 表

proce_priv 表可以对存储过程和存储函数设置操作权限。


#### 三、账户管理

> 新建用户

````
CREATE USER user_specification
[, user_specification...] ...

user_specification:
    user@host
    [
        IDENTIFIED BY [PASSWORD] 'password'
        |
        IDENTIFIED WITH auth_plugin [AS 'auth_string']
    ]

============================================

example:

CREATE USER 'jeffrey'@'localhost' IDENTIFIED BY PASSWORD '*B552D554DDD3EE421E87404BE5D195BD5E8B8777'
````

````
GRANT privileges ON db.table
TO user@host [IDENTIFIED BY 'password']
[, user [IDENTIFIED  BY 'password'] ]
[WITH GRANT OPTION]

=============================================

example

GRANT SELECT, UPDATE ON *.* TO 'testUser'@'localhost' IDENTIFIED BY  PASSWORD '*B552D554DDD3EE421E87404BE5D195BD5E8B8777'
````

````
从 user 表直接添加记录
````

> 删除用户

````
DROP USER 'user'@'host'; // 删除当前域名的用户权限
DROP USER; // 删除用户在所有域名的权限
````

````
DELETE FROM user WHERE host="hostname" and user="username"
````

> root 用户修改自己的密码

````
1) 通过 mysqladmin
mysqladmin -u username -h host -p password "newpwd"

2）操作 user 表
UPDATE user set Password=PASSWORD("rootpwd") where User="root" and Host="host"

3)使用 set 语句
SET PASSWORD = PASSWORD("rootpwd")
````

> 修改他人密码

````
1）使用 set 语句
SET PASSWORD FOR 'user'@'host' = PASSWORD('somepassword') 

2）操作 user 表
UPDATE user set Password=PASSWORD("rootpwd") where User="root" and Host="host"
````

> 普通用户修改自己密码

````
SET PASSWORD = PASSWORD("rootpwd")
````

> 加载权限表

````
FLUSH PRIVILEGES;
````