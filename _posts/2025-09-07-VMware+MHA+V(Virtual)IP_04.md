---
title: VMWare + MHA + V(Virtual)IP 04
author: B
date: 2025-09-07 17:12:00 +0900
categories: [MariaDB]
---
### 05. MHA Failover 테스트

### MHA Manager 테스트 설정 (MHA Manager)

```shell
# MHA Manager 구동
start

# MHA MAnager 상태 확인
status

# Master와 Slave의 Replication 상태 변동 확인을 위해 log 확인
log
```

<p align="center" width="100%">
    <a href="/commons/MariaDB_MHA/04/01.png" class=""><img src="/commons/MariaDB_MHA/04/01.png"></a>
</p>

### [Master] MariaDB Down

- 다운 시키기 전, Slave 측에서 Replication이 제대로 연결되어 있는지 확인
- 다운 시키면서 MHA Manager 측에서 Log로 같이 보기

```shell
service mariadb stop
```

### Manager Log 확인 (MHA Manager)
- 만일 위와 같이 `log`로 진행하지 않았다면 `vi /mha/manager.log`로 확인

<p align="center" width="100%">
    <a href="/commons/MariaDB_MHA/04/02.png" class=""><img src="/commons/MariaDB_MHA/04/02.png"></a>
</p>

<p align="center" width="100%">
    <a href="/commons/MariaDB_MHA/04/03.png" class=""><img src="/commons/MariaDB_MHA/04/03.png"></a>
</p>

```text
Phase 1: Configuration Check Phase -> 현재 살아있는 슬레이브 서버를 이용하여 마스터 서버가 죽었는지 다시 한번 체크합니다. 또한 현재 살아있는 슬레이브 서버를 체크합니다.
Phase 2: Dead Master Shutdown Phase -> (마스터 서버로 SSH 접속이 가능한 경우) 마스터의 MySQL로 접속이 불가하기 때문에 마스터로 SSH 접속하여 MySQL 데몬을 내립니다. 
Phase 3: Master Recovery Phase -> 슬레이브 서버의 릴레이 로그의 POS(포지션)을 확인하여 가장 최신의 릴레이 로그를 가진 서버를 새 마스터로 선정하며, 
기존 마스터로부터 가져온 바이너리 로그와 슬레이브 서버들의 릴레이 로그를 비교하여 최신의 데이터를 새 마스터에 적용합니다.
Phase 3.1: Getting Latest Slaves Phase
Phase 3.2: Saving Dead Master's Binlog Phase
Phase 3.3: Determining New Master Phase
Phase 3.4: New Master Diff Log Generation Phase
Phase 3.5: Master Log Apply Phase
Phase 4: Slaves Recovery Phase -> Phase 3의 작업과 동일하게 마스터 서버의 바이너리 로그와 슬레이브 서버의 릴레이 로그를 비교하여 데이터를 최신의 상태로 적용합니다.
Phase 4.1: Starting Parallel Slave Diff Log Generation Phase
Phase 4.2: Starting Parallel Slave Log Apply Phase
Phase 5: New master cleanup phase -> 새 마스터 서버의 slave 정보를 리셋합니다.
```

- 진행을 하다가 아래와 같은 Error 메세지가 나올 경우 `rm -f /mha/mha.failover.complete` 명령을 통하여 파일을 지워주어야 한다.

```text
[error][/usr/local/share/perl5/MHA/MasterFailover.pm, ln309] Last failover was done at 2024/03/22 10:02:36. Current time is too early to do failover again. If you want to do failover, manually remove /mha/mha.failover.complete and run this script again.
[error][/usr/local/share/perl5/MHA/ManagerUtil.pm, ln177] Got ERROR:  at /usr/local/bin/masterha_manager line 65.
```

### MHA 동작 확인 (Master/Slave)
- [Master] VIP 설정했던 ens34 인터페이스 비활성화 여부 확인

```shell
ifconfig
```

<p align="center" width="100%">
    <a href="/commons/MariaDB_MHA/04/04.png" class=""><img src="/commons/MariaDB_MHA/04/04.png"></a>
</p>

- [Slave] Replication 상태 확인 및 VIP 활성화 여부 확인

```shell
MariaDB > show slave status\G
MariaDB > exit

ifconfig
```
<p align="center" width="100%">
    <a href="/commons/MariaDB_MHA/04/05.png" class=""><img src="/commons/MariaDB_MHA/04/05.png"></a>
</p>

### 장애조치 완료 후 원복 작업
- 기존 Master를 NEw Slave 역할로 변경하기
- [MHA Manager] 장애 조치가 이뤄진 시점의 새 마스터 서버의 바이너리 로그 파일 및 POS(포지션)정보를 포함한 슬레이브 설정 명령어 확인

```shell
grep "All other slaves should start replication from here" /mha/manager.log
    CHANGE MASTER TO MASTER_HOST='192.168.108.102', MASTER_PORT=3306, MASTER_LOG_FILE='bin.000004', MASTER_LOG_POS=322, MASTER_USER='rep', MASTER_PASSWORD='xxx';
```

<p align="center" width="100%">
    <a href="/commons/MariaDB_MHA/04/06.png" class=""><img src="/commons/MariaDB_MHA/04/06.png"></a>
</p>

- `CHANGE MASTER TO ... MASTER_PASSWORD'xxx';` 문장 복사
    - [New Slave] MariaDB에 접속하여 복사한 문장 붙여 넣은 후 MASTER_PASSWORD='XXX' 내 XXX 만 변경하여 적용

    ```shell
    service mariadb start
    mysql -uroot -p

    MariaDB > CHANGE MASTER TO MASTER_HOST='192.168.108.102', MASTER_PORT=3306, MASTER_LOG_FILE='bin.000004', MASTER_LOG_POS=322, MASTER_USER='rep', MASTER_PASSWORD='rep';
    MariaDB > start slave;
    MariaDB > show slave status\G
    ```

    - Slave 상태 확인 시, 정상 동작 여부 확인

    <p align="center" width="100%">
        <a href="/commons/MariaDB_MHA/04/07.png" class=""><img src="/commons/MariaDB_MHA/04/07.png"></a>
    </p>

- 최종 서버 구성
<p align="center" width="80%">
    <a href="/commons/MariaDB_MHA/04/08.png" class=""><img src="/commons/MariaDB_MHA/04/08.png"></a>
</p>