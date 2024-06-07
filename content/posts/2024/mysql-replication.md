---
title: "快速搭建一个MySQL主从复制"
date: 2024-06-06T21:08:11+08:00
draft: false

categories:
- 数据库

tags:
- MySQL
- MySQL主从复制
---

# 快速搭建一个MySQL主从复制
> tip:本文所指的服务器指的是MySQL服务器
## 复制的原理

传统的复制方法是基于从主服务器的二进制日志复制事件,需要在主服务器和从服务器之间同步日志文件及其位置，属于单向异步复制。

### 复制过程涉及的线程：  
主服务器上：
1. Binlog Dump线程。当从服务器连接到主服务器时,主服务器会创建一个线程将二进制日志内容发送给从服务器。

从服务器上：
1. Replication I/O 接收线程。当在从服务器上输入 START REPLICA 语句时,会创建一个 I/O (接收) 线程,它会连接到主服务器并要求主服务器发送其二进制日志中记录的更新内容。
2. Replication SQL 线程。SQL线程是读取由Replication I/O 接收线程写入的中继日志，并执行其中包含的事务。

### 复制过程：

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202406022327905.png)

1. 数据变更会被记录到主服务器的binary log
2. 主服务器上的Binlog Dump线程将二进制日志内容发送给从服务器。
3. 从服务器的IO线程负责将从主服务器收到的变更数据写入中继日志中。
4. 从服务器的SQL线程负责读取中继日志数据,并将变更应用于从服务器数据库。

### 复制的好处：
1. 方便扩展：在复制的情况下所有的更新操作还是要在主服务器上执行，但是,读取操作可以在一个或多个从服务器上进行。所以可以通过增加从服务器的数量,大大提高读取速度。
2. 确保数据安全：从服务器可以暂停复制过程，因此可以在从服务器上运行备份服务而不会损坏相应的主数据库的数据。
3. 可用于数据分析：可以在主服务器上创建实时数据,同时在从服务器上进行数据分析,而不会影响主服务器的性能。


## 本次实验所用的开发环境如下：

```
操作系统: Ubuntu:22.04
Docker引擎：Docker Engine:26.1.3
MySQL:8.4.0(通过容器安装)
```
本次实验使用Docker Compose搭建一个主MySQL服务器两个从MySQL服务器

## 实验项目的文件结构及运行说明

```
.
├── .env
├── start.sh
├── clean.sh
├── docker-compose.yaml
├── master
│   ├── conf
│   │   └── mysqld.cnf
│   ├── data
│   ├── master.env
│   └── sql
│       └── mytest.sql
├── slave1
│   ├── conf
│   │   └── mysqld.cnf
│   ├── data
│   └── slave1.env
└── slave2
    ├── conf
    │   └── mysqld.cnf
    ├── data
    └── slave2.env
```

该项目文件可以通过[Github](Github)获取或者可以参考后面的`实验项目的具体文件内容`小节。

准备好上述实验的文件后，只需要执行start.sh文件即可启动：
```bash
./start.sh
```

执行脚本输出如下：
![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202406021835379.png)

在输出信息中查看从服务器的状态：

![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202406022022143.png)
`Replica_IO_Running` 和 `Replica_SQL_Running` 都应该为 Yes。


![](https://jsdelivr.codeqihan.com/gh/MysticalDream/images/assets/202406022024465.png)
`Last_Error`、`Last_IO_Error` 、 `Last_SQL_Error` 应该为空。

如果实验完成后想清理掉运行的服务和数据，可以运行下面的命令：
```bash
sudo ./clean.sh
```
>clean脚本包含的操作：
>1. 停止并移除容器
>2. 删除mysql的数据

注意如果shell脚本无法执行，可以尝试以下命令:
```bash
#将xxx换成对应的脚本文件名
chmod +x xxx.sh
```

### 测试

创建测试数据库

```bash
docker exec mysql_master sh -c "export MYSQL_PWD=root;mysql -uroot -e 'create database mytest'"
```

创建一个测试表并插入测试数据
```bash
docker exec mysql_master sh -c "export MYSQL_PWD=root;mysql -uroot mytest -e 'create table test(id int auto_increment,name varchar(32) not null,primary key(id));insert into test(name)values(\"zs\")'"
```

查看master中测试表的数据
```bash
docker exec mysql_master sh -c "export MYSQL_PWD=root;mysql -uroot mytest -e 'select * from test \G'"
```

查看slave1中测试表的数据
```bash
docker exec mysql_slave1 sh -c "export MYSQL_PWD=root; mysql -u root mytest -e 'select * from test \G'"
```

查看slave2中测试表的数据
```bash
docker exec mysql_slave2 sh -c "export MYSQL_PWD=root; mysql -u root mytest -e 'select * from test \G'"
```

## 实验项目的具体文件内容

对于搭建过程涉和配置详情也可以直接参考官方文档哦:
[https://dev.mysql.com/doc/refman/8.4/en/replication-howto.html](https://dev.mysql.com/doc/refman/8.4/en/replication-howto.html)

[https://dev.mysql.com/doc/refman/8.4/en/replication.html](https://dev.mysql.com/doc/refman/8.4/en/replication.html)

本次实验的主服务器的数据库是没有现有数据的，对于主服务器有数据的如果需要和从服务器同步现有数据，需要将主服务器上的数据转储到每个从数据库，可以参考官方文档的说明：
[https://dev.mysql.com/doc/refman/8.4/en/replication-snapshot-method.html](https://dev.mysql.com/doc/refman/8.4/en/replication-snapshot-method.html)

[https://dev.mysql.com/doc/refman/8.4/en/replication-setup-replicas.html#replication-howto-existingdata](https://dev.mysql.com/doc/refman/8.4/en/replication-setup-replicas.html#replication-howto-existingdata)

### 主服务器的配置

master/master.env的文件内容如下：

```
MYSQL_ROOT_PASSWORD=root
TZ=Asia/Shanghai
```

master/conf/mysqld.cnf文件内容如下：

```
[mysqld]
mysql_native_password=on

# 此变量指定服务器ID。默认情况下,server_id 被设置为 1。
# 范围从 1 到 2^32 - 1
# 在主从复制集群中必须为每个复制服务器指定一个唯一的服务器 ID
server_id = 1

# 配置用于二进制日志文件的名称前缀
# 如果没有配置log_bin,则名称前缀默认为 "binlog"
log_bin = 1

# 设置二进制日志记录格式，可以填的值有MIXED、STATEMENT、ROW
binlog_format = ROW


# https://dev.mysql.com/doc/refman/8.4/en/replication-options-binary-log.html#option_mysqld_binlog-do-db
# 指定mytest数据库不写入binlog
#binlog_do_db = mytest


# https://dev.mysql.com/doc/refman/8.4/en/replication-options-binary-log.html#option_mysqld_binlog-ignore-db
# 指定mysql数据库不写入binlog
binlog_ignore_db =mysql

```

### 从服务器的配置

#### slave1配置

slave1/slave1.env的文件内容（同master.env）如下：
```
MYSQL_ROOT_PASSWORD=root
TZ=Asia/Shanghai
```

slave1/conf/mysqld.conf的文件内容如下：

```
[mysqld]
mysql_native_password=on

# 此变量指定服务器ID。默认情况下,server_id 被设置为 1。
# 范围从 1 到 2^32 - 1
# 在主从复制集群中必须为每个复制服务器指定一个唯一的服务器 ID
server_id = 2

# 配置用于二进制日志文件的名称前缀
# 如果没有配置log_bin,则名称前缀默认为 "binlog"
log_bin = 1

# 设置二进制日志记录格式，可以填的值有MIXED、STATEMENT、ROW
#binlog_format = ROW


# https://dev.mysql.com/doc/refman/8.4/en/replication-options-binary-log.html#option_mysqld_binlog-do-db
# 指定mytest数据库不写入binlog
#binlog_do_db = mytest


# https://dev.mysql.com/doc/refman/8.4/en/replication-options-binary-log.html#option_mysqld_binlog-ignore-db
# 指定mysql数据库不写入binlog
#binlog_ignore_db =mysql

read_only=1
```

#### slave2配置

slave2/slave2.env的文件内容（同master.env）如下：
```
MYSQL_ROOT_PASSWORD=root
TZ=Asia/Shanghai
```

slave2/conf/mysqld.conf的文件内容如下：

```
[mysqld]
mysql_native_password=on

# 此变量指定服务器ID。默认情况下,server_id 被设置为 1。
# 范围从 1 到 2^32 - 1
# 在主从复制集群中必须为每个复制服务器指定一个唯一的服务器 ID
server_id = 3

# 配置用于二进制日志文件的名称前缀
# 如果没有配置log_bin,则名称前缀默认为 "binlog"
log_bin = 1

# 设置二进制日志记录格式，可以填的值有MIXED、STATEMENT、ROW
#binlog_format = ROW


# https://dev.mysql.com/doc/refman/8.4/en/replication-options-binary-log.html#option_mysqld_binlog-do-db
# 指定mytest数据库不写入binlog
#binlog_do_db = mytest


# https://dev.mysql.com/doc/refman/8.4/en/replication-options-binary-log.html#option_mysqld_binlog-ignore-db
# 指定mysql数据库不写入binlog
#binlog_ignore_db =mysql

read_only=1
```


### docker compose配置

.env的文件内容如下：
```
MYSQL_VERSION=8.4.0
```

docker-compose.yaml的文件内容如下：

```yaml
services:
  master:
    container_name: mysql_master
    image: mysql:${MYSQL_VERSION}
    env_file:
      - "./master/master.env"
    volumes:
      - "./master/data:/var/lib/mysql"
      - "./master/conf:/etc/mysql/conf.d"
  slave1:
    container_name: mysql_slave1
    image: mysql:${MYSQL_VERSION}
    env_file:
      - "./slave1/slave1.env"
    volumes:
      - "./slave1/data:/var/lib/mysql"
      - "./slave1/conf:/etc/mysql/conf.d"
  slave2:
    container_name: mysql_slave2
    image: mysql:${MYSQL_VERSION}
    env_file:
      - "./slave2/slave2.env"
    volumes:
      - "./slave2/data:/var/lib/mysql"
      - "./slave2/conf:/etc/mysql/conf.d"
```

### shell脚本

clean.sh文件的内容如下:
```bash
#!/bin/bash

docker compose down -v
rm -rf ./master/data/*
rm -rf ./slave*/data/*
```

start.sh文件的内容如下：
```bash
#!/bin/bash

# 清除旧的环境
. ./clean.sh

# 启动 Docker 容器
docker compose up -d

# 常量定义
MASTER_CONTAINER="mysql_master"
SLAVE_CONTAINERS=("mysql_slave1" "mysql_slave2")
REPLICATOR_USER="replicator"
REPLICATOR_PASSWORD="123456"
ROOT_PASSWORD="root"
MYSQL_CMD="mysql -u root"
WAIT_INTERVAL=4

# 等待 MySQL 容器启动
wait_for_mysql() {
    local container_name=$1
    until docker exec "$container_name" sh -c "export MYSQL_PWD=$ROOT_PASSWORD; $MYSQL_CMD -e ';'"
    do
        echo "Waiting for $container_name database connection..."
        sleep $WAIT_INTERVAL
    done
}

# 创建复制用户
create_replicator_user() {
    local container_name=$1
    local stmt="CREATE USER '$REPLICATOR_USER'@'%' IDENTIFIED WITH mysql_native_password BY '$REPLICATOR_PASSWORD';"
    stmt+="GRANT REPLICATION SLAVE ON *.* TO '$REPLICATOR_USER'@'%';"
    stmt+="FLUSH PRIVILEGES;"
    docker exec "$container_name" sh -c "export MYSQL_PWD=$ROOT_PASSWORD; $MYSQL_CMD -e \"$stmt\""
}

# 获取主服务器的二进制日志状态
get_master_status() {
    local container_name=$1
    docker exec "$container_name" sh -c "export MYSQL_PWD=$ROOT_PASSWORD; $MYSQL_CMD -e 'SHOW BINARY LOG STATUS\G'" | awk '/File:|Position:/{print $2}'
}

# 配置从服务器并启动复制
start_replication() {
    local slave_name=$1
    local master_log_file=$2
    local master_log_pos=$3
    local start_slave_stmt="CHANGE REPLICATION SOURCE TO SOURCE_HOST='$MASTER_CONTAINER',SOURCE_USER='$REPLICATOR_USER',SOURCE_PASSWORD='$REPLICATOR_PASSWORD',SOURCE_LOG_FILE='$master_log_file',SOURCE_LOG_POS=$master_log_pos; START REPLICA;"
    docker exec "$slave_name" sh -c "export MYSQL_PWD=$ROOT_PASSWORD; $MYSQL_CMD -e \"$start_slave_stmt\""
    echo "$slave_name replica status:"
    docker exec "$slave_name" sh -c "export MYSQL_PWD=$ROOT_PASSWORD; $MYSQL_CMD -e 'SHOW REPLICA STATUS \G'"
}

# 等待主服务器启动
wait_for_mysql "$MASTER_CONTAINER"

# 创建复制用户
create_replicator_user "$MASTER_CONTAINER"

# 获取主服务器的二进制日志状态
MS_STATUS=($(get_master_status "$MASTER_CONTAINER"))
CURRENT_LOG=${MS_STATUS[0]}
CURRENT_POS=${MS_STATUS[1]}

# 等待从服务器启动并配置复制
for slave in "${SLAVE_CONTAINERS[@]}"; do
    wait_for_mysql "$slave"
    start_replication "$slave" "$CURRENT_LOG" "$CURRENT_POS"
done
```
