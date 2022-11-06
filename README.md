# setup-master-slave-replication-MySQL-on-CentOS7


![masterslave-diagram](https://user-images.githubusercontent.com/78303835/200170582-e721e699-d753-465a-b55f-0a9a213d1c6b.jpg)

picture: [Master Slave Diagram and MVC Diagram]

## Prerequisites
1. two VM (virtual Machine) with root access
2. Working Internet

### Example
- My Computer is macOS.

I create two CentOS VM that is
- Master IP: 192.168.100.129 as [root@localhost mas]#
- Slave IP: 192.168.100.130 as [root@localhost sv]#

(config on CentOS VM with cmd: nmtui)

both VM connect with IP: 192.168.100.0 (config on VMware Fusion)


## On Master
1. access root

```[mas@localhost ~]$ su root``` result: ```[root@localhost mas] #```

2. install mariadb (MySQL)

```[root@localhost mas] # yum -y install mariadb-server```

3. enable mariadb

```[root@localhost mas] # systemctl status mariadb``` result: inactive (dead)

```[root@localhost mas] # systemctl enable mariadb```

```[root@localhost mas] # systemctl start mariadb```

```[root@localhost mas] # systemctl status mariadb``` result: active (running)

4. install net-tools on the server (opional)

```[root@localhost mas] # yum install net-tools```

5. check port3306 mysqld

```[root@localhost mas] # netstat -nltp```

```[root@localhost mas] # firewall-cmd --get-services | grep mysql --color```

```[root@localhost mas] # firewall-cmd --permanent --add-service=mysql```

```[root@localhost mas] # firewall-cmd --reload```

```[root@localhost mas] # firewall-cmd --list-all```

6. set root password mysql

```[root@localhost mas] # mysql_secure_installation```

7. log in to mariadb

```[root@localhost mas] # mysql -u root -p```
result: ```MariaDB [(none)]>```

example - query db cmd:

```MariaDB [(none)]> show schemas;```

```MariaDB [(none)]> create database db1;```

```MariaDB [(none)]> use db1;```

```MariaDB [db1]> create table dbtb1 (name varchar(40));```

```MariaDB [db1]> insert into db1.dbtb1 values('Pinky');```

```MariaDB [db1]> show schemas;```

```MariaDB [db1]> quit;``` result: ```[root@localhost mas] #```

other mysql cmd using [w3shools.com/mysql](https://www.w3schools.com/mysql/default.asp).

8. edit MySQL configuration file

```[root@localhost mas] # vi /etc/my.cnf```

on macOS
- opt+s = insert state

add server-id and log-bin below ```[mysqld]```
```
server-id=1
log-bin=mysql-bin
```

- opt+d = remove insert state
- :wq = quit and save file

restart mariadb

```[root@localhost mas] # systemctl restart mariadb```

9. create new user(s) for Slave

```[root@localhost mas] # mysql -u root -p```

```MariaDB [(none)]> grant replication slave on *.* to 'repl'@'192.168.100.130' identified by 'repl';```

[ master: 192.168.100.129,
slave: 192.168.100.130 ]

```MariaDB [(none)]> flush privileges;```

```MariaDB [(none)]> flush tables with read lock;```

```MariaDB [(none)]> show master status;```

```MariaDB [(none)]> unlock tables;```

10. copy data from master to slave

```[root@localhost mas] # mysqldump -uroot -p db1 > db1.sql```

```[root@localhost mas] # ls``` result: db1.sql

```[root@localhost mas] # scp db1.sql root@192.168.101.128:/tmp``` result: downloaded db1.sql


## On Slave
1.-6. On Master

7. import data from master

```[root@localhost sv] # mysql -uroot -p -e "create database db1"```

```[root@localhost sv] # mysql -uroot -p db1 < /tmp/db1.sql```

```[root@localhost sv] # mysql -uroot -p -e "show schemas"```

8. edit MySQL configuration file

```[root@localhost sv] # vi /etc/my.cnf```

on macOS
- opt+s = insert state

add server-id and log-bin below ```[mysqld]```
```
server-id=2
replicate-wild-do-table=db1.%
```

- opt+d = remove insert state
- :wq = quit and save file

restart mariadb

```[root@localhost sv] # systemctl restart mariadb```

```[root@localhost sv] # ssystemctl status mariadb```

9. change to master...

```[root@localhost sv] # mysql -u root -p```

```
MariaDB [(none)]> change to master
->master_host='192.168.100.129',
->master_user='repl',
->master_password='repl'.
->master_log_file='mysql-bin.000001',
->master_log_pos=245;
```

```MariaDB [(none)]> start slave;```

```MariaDB [(none)]> show slave status;```

```MariaDB [(none)]> show slave status\G;```

result:

-Slave_IO_State: Waiting for master to send event

-Slave_IO_Running: Yes

-Slave_SQL_Running: Yes

-no errors

```MariaDB [(none)]> stop slave``` (optional)


### Finished to connect master slave. they can apply separate db or mix together depend on deployment of other system.
