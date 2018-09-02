### 一、数据备份

数据备份是数据库管理员非常重要的工作之一。系统意外崩溃活着硬件的损坏都可能导致数据库的丢失，因此 mysql 管理员应该定期地备份数据库，使得在意外情况发生时，尽可能减少损失。

> 使用 mysqldump 命令备份      

mysqldump 是 mysql 提供的一个非常有用的数据库备份工具。MySQLdump 命令执行时，可以将数据库备份成一个文本文件，该文件中实际包含了多个 CREATE 和 INSERT 语句，使用这些语句可以重新创建表和插入数据。

````
// 备份整个数据库
mysqldump -uuser -h host -p dbname  > filename.sql

// 备份单个个数据表
mysqldump -uuser -h host -p dbname [tbname, [tbname...]] > filename.sql

// 备份多个数据库
mysqldump -uuser -h host -p --databases dbname, [dbname, [dbname...]] > filename.sql

// 备份所有数据库
mysqldump -uuser -h host -p --all-databases > filename.sql

````

> 直接复制整个数据库目录

因为 mysql 表保存为文件方式，所以可以直接复制 mysql 数据库的存储目录及文件进行备份。在不同的平台下，mysql 的数据库目录位置不一定相同。

这是一种简单、快速、有效的备份方式。想要保持备份的一致性，备份前需要对相关表执行 LOCK TABALES 操作，然后对表执行 FLUSH TABLES 。这样当复制数据库目录中的文件时，允许其他客户继续查询表。需要 FLUSH TABLES 语句来确保开始备份前将所有激活的索引页写入硬盘。当然，也可以停止 mysql 服务在进行备份操作。

这种方法虽然简单，但并不是最好的方法。因为这种方法对 InnoDB 存储引擎的表不适用。使用这种方法备份的数据最好恢复到相同版本的服务器中，不同的版本可能不兼容。

> 使用 mysqlhotcopy 工具快速备份

mysqlhotcory 是一个 perl 脚本。它使用 LOCK TABLES、FLUSH TABLES 和 cp 或 scp 来快速备份数据库。他是备份数据库或单个表的最快的途径，但是它只能运行在数据库目录所在的机器上，并且只能备份 MyISAM 类型的表。mysqlhotcopy 只能在 unix 系统中运行。

mysqlhotcopy 只是将表所在的目录复制到另一个位置，只能用于备份 MyISAM 和 ARCHIVE 表。备份 InnoDB 类型的数据表时会出现错误信息。由于它复制本地格式的文件，故也不能移植到其他硬件或操作系统下。


### 二、数据恢复

管理人员操作的失误、计算机故障以及其他意外情况，都会导致数据的丢失和破坏。当数据丢失或意外破坏时，可以通过恢复已经备份的数据尽量减少数据丢失和破坏造成的损失。

> 使用 mysql 命令恢复

````
// 未登录 mysql
mysql -uuser -p [dbname] < filename.sql

// 登录 mysql
source filename.sql
````

> 直接复制数据库目录

通过这种方式恢复时，必须保存数据的数据库和待恢复的数据库服务器的主版本相同。而且这种方式只对MyISAM 引擎的表有效，对于 InnoDB 引擎的表不可用。

执行恢复以前关闭 mysql 服务，将备份的文件或目录覆盖 mysql 的 data 目录，启动 mysql 服务。对于 Linux/Unix 操作系统来说，复制完文件需要将文件的用户和组更改为 mysql 运行的用户和组，通常用户是 mysql，组也是 mysql。

> mysqlhotcopy 快速恢复


### 三、数据库迁移

数据库迁移就是把数据从一个系统迁移到另一个系统上。数据迁移有一下原因：

1、需要安装新的数据库服务器     

2、mysql 版本更新     

3、数据库管理系统的变更（如从 MicrosoftSQL server 迁移到 mysql）

> 相同版本的 mysql 数据库迁移

相同版本的 mysql 数据库之间的迁移就是在主版本号相同的 mysql 数据库之间进行数据库移动。迁移过程其实就是在源数据库备份和目标数据库恢复过程组合。

在讲解数据库备份和恢复时，已经知道最简单的方式是通过复制数据库文件目录，但是此种方法只适用于 MyISAM 引擎的表，因此最常用和最安全的方式是使用 mysqldump 命令导出数据，然后在目标数据库服务器使用 mysql 命令导入。

> 不同版本的 mysql 数据库之间的迁移

因为数据库升级等原因，需要将较旧版本的 mysql 数据库中的数据迁移到较新版本的数据库中。mysql服务器升级时，需要先停止服务，然后卸载旧版本，并安装新版的mysql。这种更新方法很简单，如果想保留旧版本中的用户访问控制信息，则需要备份 mysql 中的 mysql 数据库，在新版本安装完成中，重新读入 mysql 备份文件中的信息。

旧版本与新版本的 mysql 可能使用不同的默认字符集，例如 mysql 4.x 中大多使用 latinl 作为默认字符集，而 mysql 5.x 的默认字符集为 utf8。如果数据库中有中文数据，迁移过程中需要对默认字符集进行修改，不然可能无法正常显示结果。

新版本会对旧版本有一定的兼容性。从旧版本的 mysql 向新版本 mysql 中迁移时，对于 MyIsAM 引擎的表，可以直接复制数据库文件，也可以使用 mysqldump 工具。对于 InnoDB 引擎的表，一般只能使用 mysqldump 将数据导出。然后使用 mysql 命令导入到目标服务器上。

> 不同数据库之间的迁移

不同类型的数据库之间的迁移，是指把 mysql 的数据库转移到其他类型的数据库，例如从 mysql 迁移到 oracle。

迁移之前，需要了解不同数据库的架构，比较他们之间的差异。不同数据库中定义相同类型的数据的关键字可能会不同。例如 mysql 中日期字段分为 DATE 和 TIME 两种。而 oracle 日期字段只有 DATE。


### 四、表的导出和导入

有时侯需要将 mysql 数据库中的数据导出到外部存储文件中，mysql 数据库中的数据可以导出成 sql 文本文件、xml 文件或者 html 文件。同样这些导出文件也可以导入到 mysql 数据库中。

> 使用 select ... INTO OUTFILE 导出文本文件

````
SELECT columnlist FROM table WHERE codition INTO OUTFILE 'filename'
[OPTIONS]

--OPTIONS 选项
  FIELDS   TERMINATED            BY 'value'
  FIELDS   [OPTIONALLY] ENCLOSED BY 'value'
  FIELDS   ESCAPED               BY 'value'
  LINES    STARTING              BY 'value'
  LINES    TERMINATED            BY 'value'
````

SELECT ... INTO OUTFILE 语句可以快速地把表转储到服务器上。如果想要在服务器主机之外的部分客户主机上创建结果文件，不能使用 SELECT ... INTO OUTFILE。在这种情况下，应该在客户主机上使用比如 “ Mysql -e "SELECT ..." > filename ” 的命令，来生成文件。

> 使用 mysqldump 命令导出文本文件

````
mysqldump -T path -uuser -p dbname [tables] [OPTIONS]

--OPTIONS 选项
--fields-terminated-by=value
--fields-enclosed-by=value
--fields-optionally-enclosed-by=value
--fields-escaped-by=value
--fields-terminated-by=value
````

> 使用 mysql 命令导出文本文件

````
mysql -uuser -p --execute = "SELECT 语句" dbname > filename.txt
````

> 使用 LOAD DATA INFILE 方式导入文本文件

````
LOAD DATA INFILE 'filename.txt' INTO TABLE tablename [OPTIONS] [IGNORE number LINES]

-- OPTIONS 选项
FIELDS TERMINATED BY 'value'
FIELDS [OPTIONALLY] ENCLOSED BY 'value'
FIELDS ESCAPED BY 'value'
LINES STARTING BY 'value'
LINES TERMINATED BY 'value'
````

> 使用mysqlimport 命令导入数据

````
mysqlimport -uuser -p dbname filename.txt [OPTIONS]

--OPTIONS 选项
--fields-terminated-by=value
--fields-enclosed-by=value
--fields-optionally-enclosed-by=value
--fields-escaped-by=value
--lines-terminated-by=value
--ignore-lines=n
````
dbname 为导入的表所在的数据库名称。注意，mysqlimport 命令不指定导入数据库的表名称，数据表的名称有导入文件名称确定，即文件名作为表名，导入数据之前该表必须存在。

