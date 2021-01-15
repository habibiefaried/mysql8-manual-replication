# mysql8-manual-replication
MySQL8 master-slave replication with manual install method

# Install

1. On host

```
docker-compose up -d
```

2. On **container master**

```
# docker exec -it 13087d4ed566 bash
bash-4.2# mysql -uroot -prootpass
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 13
Server version: 8.0.22 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CREATE USER 'repl'@'%' IDENTIFIED BY 'slavepass';
Query OK, 0 rows affected (0.02 sec)

mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
Query OK, 0 rows affected (0.01 sec)

mysql> SHOW MASTER STATUS;
+--------------------+----------+--------------+------------------+-------------------+
| File               | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+--------------------+----------+--------------+------------------+-------------------+
| mysql-bin-1.000003 |      659 |              |                  |                   |
+--------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

3. On **container replica**

```
# mysql -uroot -prootpass
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 17
Server version: 8.0.22 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CHANGE MASTER TO MASTER_HOST='master', MASTER_USER='repl',MASTER_PASSWORD='slavepass', MASTER_LOG_FILE='mysql-bin-1.000003';
Query OK, 0 rows affected, 3 warnings (0.03 sec)

mysql> START SLAVE;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> SHOW SLAVE STATUS\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: master
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin-1.000003
```

4. On **container master**

```
mysql> CREATE DATABASE HABIBIETEST; SHOW DATABASES;
Query OK, 1 row affected (0.01 sec)

+--------------------+
| Database           |
+--------------------+
| HABIBIETEST        |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)
```

Check whether it's replicated on client node

## Prove it's persistence

1. Issue `docker-compose down`

2. Issue `docker-compose up`

## Result?

No, it's failing due to the position log was changed. The replication won't be working after restart. However, the data still can be accessed