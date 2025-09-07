---
title: VMWare + MHA + V(Virtual)IP 03
author: B
date: 2025-09-07 16:17:00 +0900
categories: [MariaDB]
---
## 4. MHA 설치 및 세팅

### MHA 관련 패키지 설치 (모든 서버)
```shell
yum -y install epel perl-devel perl-CPAN perl-DBD-MySQL perl-Config-Tiny perl-Log-Dispatch perl-Parallel-ForkManager perl-Module-Install -y
```

### MHA에서 사용할 디렉토리 생성 (모든 서버)

```shell
mkdir /mha
```

### 유저 생성 밑 권한 설정 (MHA Manager 서버)

```shell
useradd -g mysql -d /home/mhauser -m -s /bin/bash mhauser
passwd mhauser
    새 암호 : [PASSWORD]
    새 암호 재입력 : [PASSWORD]
chown -R mhauser:mysql /mha
```

### MHA node 설치 및 FTP 설치 (모든 서버)

```shell
mkdir /source
cd /source
```

```shell
yum install vsftpd -y
vsftpd -v
service vsftpd start 
service vsftpd status

# 21번 포트 LISTEN 상태 확인
netstat -tlpn

# 방화벽에 ftp OPEN
firewall-cmd --add-service=ftp --permanent
firewall-cmd --reload

# 방화벽 상태 조회
firewall-cmd --list-all

# 시스템 재부팅 시에도 vsftpd 자동 시작 설정
chkconfig vsftpd on

# 등록 확인
systemctl list-unit-files | grep vsftpd

# FTP Client 측에서 root로 접근 가능하도록 설정
# 아래 파일들에서 root를 주석처리한다.
vi /etc/vsftpd/ftpusers
vi /etc/vsftpd/user_list
service vsftpd restart
```

- Filezilla를 이용하여 아래와 같이 Local에 존재하는 MHA 관련 파일을 보내둔다.
    - MHA Manager Service 에는 Node와 Manager 파일 모두 보내두고, Master와 Slave에는 동일 경로에 Node 파일만 보내둔다.
<p align="center" width="80%">
    <a href="/commons/MariaDB_MHA/03/01.png" class=""><img src="/commons/MariaDB_MHA/03/01.png"></a>
</p>

```shell
ll /source
tar xvzf mha4mysql-node-0.57.tar.gz

cd /source/mha4mysql-node-0.57
perl Makefile.PL
make && make install
```

<p align="center" width="80%">
    <a href="/commons/MariaDB_MHA/03/02.png" class=""><img src="/commons/MariaDB_MHA/03/02.png"></a>
</p>

### MHA Manager 설치 (MHA Manager 서버)

```shell
cpan YAML
    Would you like to configure as much as possible automatically? [yes] yes
    Would you like me to automatically choose some CPAN mirror sites for you? (This means connecting to the Internet) [yes] yes

perl -MCPAN -e "install File::Remove"
perl -MCPAN -e "install Build"
perl -MCPAN -e "install Module::Install" 
perl -MCPAN -e "install Net::Telnet"
perl -MCPAN -e "install Log::Dispatch"

cd /source
tar xvzf mha4mysql-manager-0.57.tar.gz

cd /source/mha4mysql-manager-0/57
perl Makefile.PL
make && make install
```

### SSH 연결 (모든 서버)
- SSH 연결 MHA 모니터링과 Failover를 수행하기 위해선 각각의 서버들이 서로 간에 비밀번호 없이 SSH에 접속할 수 있어야 한다.
- 각 서버의 SHS 계정은 아래와 같이 진행 예정이다.
    - MHA Manager : mhauser
    - Master/Slave : root

```shell
# MHA Manager 서버
visudo
    # -- 맨 아래에 추가
    mhauser ALL = (ALL) NOPASSWD:/sbin/ifconfig

# 키생성
su - mhauser
ssh-keygen -b 2048 -t rsa -f ~/.ssh/id_rsa -q -N ""
ssh-copy-id root@192.168.108.101 
ssh-copy-id root@192.168.108.102

# Master
# -- 키생성
ssh-keygen -b 2048 -t rsa -f ~/.ssh/id_rsa -q -N "" 
ssh-copy-id mhauser@192.168.108.100  
ssh-copy-id root@192.168.108.102

# Slave
# -- 키생성
ssh-keygen -b 2048 -t rsa -f ~/.ssh/id_rsa -q -N "" 
ssh-copy-id mhauser@192.168.108.100 
ssh-copy-id root@192.168.108.101

# MHA Manager 서버
# -- 연결 테스트 (비밀번호 없이 바로 들어가져야함)
ssh root@192.168.108.101
ssh root@192.168.108.102

# Master 서버
# -- 연결 테스트 (비밀번호 없이 바로 들어가져야함)
ssh mhauser@192.168.108.100
ssh root@192.168.108.102

# Slave 서버 
# -- 연결 테스트 (비밀번호 없이 바로 들어가져야함)
ssh mhauser@192.168.108.100
ssh root@192.168.108.101

# MHA Manager 서버
# -- 파일의 내용이 변경되지 않도록 권한 수정
chmod 400 /home/mhauser/.ssh/authorized_keys

# Master/Slave 
# -- 파일의 내용이 변경되지 않도록 권한 수정
chmod 400 ~/.ssh/authorized_keys
```

### MHA Manager 커스텀 마이징

```shell
vi ~/.bash_profile
# 아래 내용 추가
    set -o vi
    alias sshcheck='/usr/local/bin/masterha_check_ssh --conf=/etc/mha.cnf'
    alias replcheck='/usr/local/bin/masterha_check_repl --conf=/etc/mha.cnf'
    alias start='nohup /usr/local/bin/masterha_manager --conf=/etc/mha.cnf &'
    alias status='/usr/local/bin/masterha_check_status --conf=/etc/mha.cnf'
    alias log='tail -f /mha/manager.log'

source ~/.bash_profile

# config 파일 생성 (root에서 진행)
cp /source/mha4mysql-manager-0.57/samples/conf/app1.cnf /etc/mha.cnf

vi /etc/mha.cnf
    # 기존 정보 모두 지운 뒤, 아래 내용 추가
    # 추가할 때, Master와 Slave IP 변경
    [server default]
    user=mha
    password=mha

    # ssh_user=mhauser
    ssh_user = root

    repl_user=rep
    repl_password=rep
    manager_workdir=/mha
    manager_log=/mha/manager.log
    remote_workdir=/mha
    master_binlog_dir=/var/lib/mysql
    master_ip_online_change_script=/mha/scripts/master_ip_online_change
    master_ip_failover_script=/mha/scripts/master_ip_failover

    [server1]
    hostname=192.168.108.101 
    candidate_master=1

    [server2]
    hostname=192.168.108.102 
    candidate_master=1
```

- Slave IP ↔ Master IP 인계하기 위한 스크립트 작성 (MHA Manager 서버)

```shell
su - mhauser
mkdir /mha/scripts

vi /mha/scripts/change_mtos.sh
    # 아래 내용 붙여넣기
    ssh root@192.168.108.101 /sbin/ifdown ens34
    sleep 3
    ssh root@192.168.108.102 /sbin/ifup ens34

vi /mha/scripts/change_stom.sh
    # 아래 내용 붙여넣기
    ssh root@192.168.108.102 /sbin/ifdown ens34 
    sleep 3
    ssh root@192.168.108.101 /sbin/ifup ens34

exit
chmod 755 /mha/scripts/change_*
cp /source/mha4mysql-manager-0.57/samples/scripts/master_ip_failover /mha/scripts/
cp /source/mha4mysql-manager-0.57/samples/scripts/master_ip_online_change /mha/scripts/

vi /mha/scripts/master_ip_failover

# 86번 라인 "## Creating an app user on the new master" 라인 밑에 있는 4줄 주석
     86       ## Creating an app user on the new master
     87       # print "Creating app user on the new master..\n";
     88       # FIXME_xxx_create_user( $new_master_handler->{dbh} );
     89       # $new_master_handler->enable_log_bin_local();
     90       # $new_master_handler->disconnect();

# 92번 라인 "## Update master ip on the catalog database, etc" 밑에 "FIXME_xxx;" 주석처리하고 94~99 내용 붙여넣기 
     92       ## Update master ip on the catalog database, etc
     93       # FIXME_xxx;
     94       if($new_master_ip eq "192.168.108.101"){
     95         system("/bin/sh /mha/scripts/change_stom.sh");
     96         }
     97       elsif($new_master_ip eq "192.168.108.102"){
     98         system("/bin/sh /mha/scripts/change_mtos.sh");
     99         }
    100
    101       $exit_code = 0;

vi /mha/scripts/master_ip_online_change

# 152번 라인 "FIXME_xxx_drop_app_user($orig_master_handler);" 주석
    152       # FIXME_xxx_drop_app_user($orig_master_handler);

# 244번 라인 "## Creating an app user on the new master" 밑에 있는 4줄 주석
    244       ## Creating an app user on the new master
    245       # print current_time_us() . " Creating app user on the new master..\n";
    246       # FIXME_xxx_create_app_user($new_master_handler);
    247       # $new_master_handler->enable_log_bin_local();
    248       # $new_master_handler->disconnect();

# 250번 라인 "## Update master ip on the catalog database, etc" 밑에 251~256 내용 붙여 넣기
    250       ## Update master ip on the catalog database, etc
    251       if($new_master_ip eq "192.168.108.101"){
    252         system("/bin/sh /mha/scripts/change_stom.sh");
    253         }
    254       elsif($new_master_ip eq "192.168.108.102"){
    255         system("/bin/sh /mha/scripts/change_mtos.sh");
    256         }
    257       $exit_code = 0;
```

### 설정 확인
```shell
su - mhauser

sshcheck
```
<p align="center" width="80%">
    <a href="/commons/MariaDB_MHA/03/03.png" class=""><img src="/commons/MariaDB_MHA/03/03.png"></a>
</p>
```shell
replcheck
```
<p align="center" width="80%">
    <a href="/commons/MariaDB_MHA/03/04.png" class=""><img src="/commons/MariaDB_MHA/03/04.png"></a>
</p>

### 참고 URL
- https://engmisankim.tistory.com/61
- https://onestone-note.tistory.com/34#3.%203.%C2%A0%20MariaDB%20Master%20/%20Slave%20%EA%B5%AC%EC%84%B1