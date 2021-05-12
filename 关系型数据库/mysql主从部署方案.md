# mysql主从部署步骤

## 部署配置主库master

- docker部署mysql主库。

```shell
docker run --name mysql-master --privileged=true -v /home/mysql/master-data:/var/lib/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:latest
```

- 进入容器内部，修改etc/mysql/my.cnf文件。

```shell
docker exec -it mysql-master /bin/bash
vim etc/mysql/my.cnf
```

修改文件如下：

```
[mysqld]
## 同一局域网内注意要唯一
server-id=100  
## 开启二进制日志功能，可以随便取（关键）
log-bin=mysql-bin
```

重启容器。

- 创建数据同步用户，授予用户 slave REPLICATION SLAVE权限和REPLICATION CLIENT权限，用于在主从库之间同步数据。

```mysql
CREATE USER 'test'@'%' IDENTIFIED BY '123456';

GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'test'@'%';
```

- 获取master的binlog状态。

执行mysql命令

```mysql
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000014 |      154 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

需要记住File和Position的属性的值。

## 部署配置从库slave

- docker部署mysql从库。

```shell
docker run --name mysql-slave --privileged=true -v /home/mysql/slave-data:/var/lib/mysql -p 3307:3306 --link mysql-master:master -e MYSQL_ROOT_PASSWORD=123456 -d mysql:latest
```

- 进入容器内部，修改etc/mysql/my.cnf文件。

```shell
docker exec -it mysql-master /bin/bash
vim etc/mysql/my.cnf
```

修改文件如下：

```
[mysqld]
## 设置server_id,注意要唯一
server-id=101  
## 开启二进制日志功能，以备Slave作为其它Slave的Master时使用
log-bin=mysql-slave-bin   
## relay_log配置中继日志
relay_log=edu-mysql-relay-bin  
```

重启容器

- 更换当前服务的状态，使其能够连接上master服务器，并且复制其数据。

```mysql
change master to master_host='master', master_user='test', master_password='123456', master_port=3306, master_log_file='mysql-bin.000014', master_log_pos=154, master_connect_retry=30;
```

命令说明：

**master_host** ：Master的地址，指的是容器的独立ip,可以通过`docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器名称|容器id`查询容器的ip

**master_port**：Master的端口号，指的是容器的端口号

**master_user**：用于数据同步的用户

**master_password**：用于同步的用户的密码

**master_log_file**：指定 Slave 从哪个日志文件开始复制数据，即上文中提到的 File 字段的值

**master_log_pos**：从哪个 Position 开始读，即上文中提到的 Position 字段的值

**master_connect_retry**：如果连接失败，重试的时间间隔，单位是秒，默认是60秒

- 开始主从复制并查看主从状态。

```mysql
mysql> start slave;
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: master
                  Master_User: test
                  Master_Port: 3306
                Connect_Retry: 30
              Master_Log_File: mysql-bin.000014
          Read_Master_Log_Pos: 154
               Relay_Log_File: 6774ae81bc25-relay-bin.000034
                Relay_Log_Pos: 367
        Relay_Master_Log_File: mysql-bin.000014
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
                         ....
             Master_Server_Id: 1
                  Master_UUID: 8976b929-bc8b-11e9-bab3-0242ac110005
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
                         ....
1 row in set (0.00 sec)
```

`Slave_IO_Running`和`Slave_SQL_Running`的值代表了主从服务之间io通信的状态和sql执行的状态，两者都为yes则表示主从复制状态正常。