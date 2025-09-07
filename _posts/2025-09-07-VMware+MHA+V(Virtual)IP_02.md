---
title: VMWare + MHA + V(Virtual)IP 02
author: B
date: 2025-09-07 10:12:00 +0900
categories: [MariaDB]
---
## 3. MariaDB Setting (모든 서버)

### MariaDB 설치

```shell
vi /tec/yum.repos.d/MariaDB.repo

# 아래 내용 붙여넣기 후 저장
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.4/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1

yum install MariaDB -y
```

### 정상 설치 여부 확인

```shell
rpm -qa | grep MariaDB
or
mariadb --version
```
<p align="center">
    <a href="/commons/MariaDB_MHA/02/01.png" class=""><img src="/commons/MariaDB_MHA/02/01.png" width="80%"></a>
</p>


### MariaDB root 패스워드 변경

```shell
service mariadb start
service mariadb status

mysql_secure_installation
    Enter current password for root (enter for none): enter
    Switch to unix_socket authentication [Y/n] y
    Change the root password? [Y/n] y
    New password: [PASSWORD]
    Re-enter new password : [PASSWORD]
    Remove anonymous users? [Y/n] y
    Disallow root login remotely? [Y/n] n
    Remove test database and access to it? [Y/n] n
    Reload privilege tables now? [Y/n] y
```

### Root 접속 테스트

```shell
mysql -uroot -p

MariaDB > show global variables like '%dir%';
```
<p align="center">
    <a href="/commons/MariaDB_MHA/02/02.png" class=""><img src="/commons/MariaDB_MHA/02/02.png" width="80%"></a>
</p>

### Replication 유저 생성 (Master/Slave)

```sql
MariaDB > create user `rep`@`%` identified by 'rep';
MariaDB > grant replication slave on *.* to 'rep';
MariaDB > flush provileges;
```
<p align="center">
    <a href="/commons/MariaDB_MHA/02/03.png" class=""><img src="/commons/MariaDB_MHA/02/03.png" width="80%"></a>
</p>

### MHA 유저 생성 (Master/Slave)
```sql
MariaDB > grant all privileges on *.* to `mha`@`%` identified by 'mha';
MariaDB > flush privileges;
```
<p align="center">
    <a href="/commons/MariaDB_MHA/02/04.png" class=""><img src="/commons/MariaDB_MHA/02/04.png" width="80%"></a>
</p>

### 설정을 위해 DB 종료 (Master/Slave)
```sql
MariaDB > exit
service mariadb stop
service mariadb status
```
<p align="center">
    <a href="/commons/MariaDB_MHA/02/05.png" class=""><img src="/commons/MariaDB_MHA/02/05.png" width="80%"></a>
</p>

### 환경 변수 설정 (Master/Slave)

```shell
vi /etc/my.cnf.d/server.cnf
```

- 파일 내 기본 설정 모두 제거 후 각 서버마다 아래 내용 붙여넣기 (* server_id는 다르게 지정)

```text
// Master DB
[mariadb]
bind-address = 0.0.0.0
port = 3306

server_id = 101
log-bin = /var/lib/mysql/bin.log
binlog_cache_size = 2M
max_binlog_size = 512M
expire_logs_days = 1

// Slave DB
[mariadb]
bind-address = 0.0.0.0
port = 3306

server_id = 2
log-bin = /var/lib/mysql/bin.log
read_only = 1
relay_log_purge = 0 
expire_logs_days = 1
```

### DB 재실행 (Master/Slave)

```shell
systemctl start mariadb
systemctl status mariadb

# 모든 서버
-- 시스템 재부팅시에도 mariadb 자동 시작 설정
systemctl enable mariadb
(or chkconfig mariadb on)

-- 등록 확인
systemctl list-unit-files | grep mariadb
```

<p align="center">
    <a href="/commons/MariaDB_MHA/02/06.png" class=""><img src="/commons/MariaDB_MHA/02/06.png" width="80%"></a>
</p>

### 방화벽 설정 (Master/Slave)   
- [Slave] 서버에서 마스터 측에 telnet 연결을 진행해본다.

```shell
yum install telnet
telnet 192.168.108.101 3306
```

- 현재 각 서버마다 3306 서버에 대한 방화벽이 오픈되어있지 않은 상태이므로 아래와 같은 메시지가 출력될 것이다.

<p align="center">
    <a href="/commons/MariaDB_MHA/02/07.png" class=""><img src="/commons/MariaDB_MHA/02/07.png" width="80%"></a>
</p>

```shell
service firewalld status
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
firewall-cmd --list-all
```

### Replication 설정 (Master/Slave)
- [Master] 서버에서 로그 파일과 위치 확인

```shell
MariaDB > show master status;
```

<p align="center">
    <a href="/commons/MariaDB_MHA/02/08.png" class=""><img src="/commons/MariaDB_MHA/02/08.png" width="80%"></a>
</p>

- [Slave] 서버에서 Replication 세팅

```shell
MariaDB > CHANGE MASTER TO master_host='192.168.108.101', master_user='rep', master_password='rep', master_port=3306, master_log_File='bin.000001', master_log_pos=332, master_connect_retry=10;
MariaDB > start slave;
MariaDB > show slave stataus\G
```

## 참고 URL
- https://da2uns2.tistory.com/entry/MariaDB-CentOS7%EC%97%90-MariaDB-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0