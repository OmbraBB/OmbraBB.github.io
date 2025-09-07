---
title: VMWare + MHA + V(Virtual)IP 01
author: B
date: 2025-09-07 10:12:00 +0900
categories: [MariaDB]
---
### MHA(Master High Availablility) 개요 :
- Master DB 서버에 장애가 발생했을 때, 이를 자동으로 감지하고 Slave DB 서버를 새로운 Master로 승격시켜 서비스 다운타임을 최소화하는 오픈 소스 고가용성(High Availability) 자동 장애 조치(Auto Failover) 솔루션이다.
- MHA 매니저 서버, 마스터 서버, 슬레이브 서버로 구성되어 데이터 복제를 통해 마스터 서버의 고장을 자동으로 탐지하고 새로운 마스터를 승격시켜 서비스 중단을 막는 역할을 한다.

### 테스트 환경
- Centos 7 minimal
- VMware Workspace 17 Player
- Network Adapter : NAT

![Desktop View](/commons/MariaDB_MHA/01/01.png)

## 1. VMware Setting (모든 서버)
### VMware 생성
- ISO 파일 `CentOS-7-x86_64-Minimal-2009.iso` 파일로 설정

![Desktop View](/commons/MariaDB_MHA/01/02_03.png)

- VM 이름 지정

![Desktop View](/commons/MariaDB_MHA/01/04_05.png)

- 가상 Disk 설정
    - Store virtual disk as a single file : 가상 디스크를 하나의 파일로 설정
    - Split virtual disk into nultiple files : 가상 디스크를 2GB 씩 분할하여 여러 파일로 설정

![Desktop View](/commons/MariaDB_MHA/01/06_07.png)
- 설정이 모두 완료된 요약본 확인
- 제대로 설정되어 있을 경우 `Power on this virtual ... ` 체크박스를 해제하여 Finish를 눌렀을 때, 해당 가상 서버가 바로 구동되지 않도록 한다. (*추가 설정 작업이 아직 남아있으므로 .. )
- Finish를 누르면, 가상 머신이 생성되었음을 확인할 수 있다.
- `Edit virtual machine ... `을 클릭하여 나머지 세팅을 진행한다.

![Desktop View](/commons/MariaDB_MHA/01/08.png)
![Desktop View](/commons/MariaDB_MHA/01/09.png)
- Memory는 로컬 환경에 맞추어 가상 머신 구동 시에도 로컬에 영향이 없을 정도인 2GB로 지정해주고, 네트워크는 NAT로 사용한다.
- 마스터와 슬레이브 서버에서만 VIP를 설정할 수 있는 Network Adapter를 하나 더 추가해주고 동일하게 NAT로 지정한다.
- 그 후 모든 작업이 끝났으면 Finish를 누른 후 가상 서버를 Play 한다.

![Desktop View](/commons/MariaDB_MHA/01/11_12.png)
- 언어를 설정해 준 후, 수동 파티션 설정을 진행한다.

![Desktop View](/commons/MariaDB_MHA/01/13.png)
- 수동 파티션 설정 (* 환경에 따라 유동적으로 변경)
    - /boot = 200MiB .. ext4
    - /var = 7GiB .. ext4
    - swap = 4GiB .. swap (* RAM 용량에 맞추기)
    - /tmp = 5GiB .. ext4
    - / = 나머지 .. ext4

![Desktop View](/commons/MariaDB_MHA/01/14_15.png)
- 설정이 완료된 후 [완료] 버튼을 누르면 변경 요약 팝업창이 뜬다. 확인 후 정상 적용 되었을 경우 [변경 사항 적용] 버튼을 누른다.
- 설치 요약 내, 모든 설정이 완료되었으면 [설치 시작]을 누른다.
- 네트워크 또한 이곳에서 설정이 가능하지만, CLI로 진행할 예정이다.

![Desktop View](/commons/MariaDB_MHA/01/16.png)
- Root 암호까지 설정이 되었으면, 재부팅을 해준다.

## 2. Network 설정 (모든 서버)
- 이 전 로컬 PC의 VMware Network 설정이 되어있는지 확인.
    - http://logforlog.tistory.com/entry/Linux-%EA%B5%AC%EC%84%B1-%ED%99%98%EA%B2%BD-Network-Setting
- vmnetcfg.exe 설치
    - 먼저 VMware Player 15.5.7 버전의 vmnetcfg를 설치한다.
    - (vmnetcfg 15.5.7 버전은 VMware Player 16.x, 17.x 버전에서도 사용이 가능하다)
    - https://www.tobias-hartmann.net/2018/12/download-vmnetcfg-exe-fuer-vmware-workstation-15-x-player/
- 다운 받은 압축 파일을 해제한 뒤, VMware 디렉토리로 옮겨준다.

![Desktop View](/commons/MariaDB_MHA/01/18.png)
- 네트워크 설정을 변경하기 위해서 [관리자 권한으로 실행]으로 프로그램을 실행한다.

![Desktop View](/commons/MariaDB_MHA/01/19.png)


- /etc/sysconfig/network-scripts/ifcfg-** 파일 설정
    - 마스터/슬레이브의 경우에는 MHA 서버와 동일한 인터페이스 ID를 가진 파일을 변경해준다.
    - ifcfg-lo는 로컬호스트 관련 설정이므로 변경 X
    - Rocky_linux9의 경우에는 /etc/NetworkManager/system-connections/ens***-nmconnection을 사용한다. (* 참고 : https://tech.viaweb.co.kr/OS/177)
<p align="center" width="100%">
    <a href="/commons/MariaDB_MHA/01/20.png" class=""><img src="/commons/MariaDB_MHA/01/20.png" width="50%"></a>
</p>


- 설정이 끝났으면, `service network restart`로 네트워크를 재시작해준 뒤, `ping 8.8.8.8`로 외부 통신이 진행되는지 확인한다.

![Desktop View](/commons/MariaDB_MHA/01/21.png)


### 기본 패키지 설치
```shell
yum install epel-release -y
yum install net-tools sysstat wget lrzsz lsof htop iftop rsync bzip2 unzip patch syslog -y
```

### 네트워크 설정 확인
```shell
ifconfig
netstat -ntlp
```
![Desktop View](/commons/MariaDB_MHA/01/22.png)
![Desktop View](/commons/MariaDB_MHA/01/23.png)

- putty를 이용하여 각 서버에 접근한다.

### VIP 설정 (Master/Slave)
- VIP의 경우 같은 IP를 사용할 것이므로 설정 후 down 시켜 줄 Slave 먼저 설정 후 ifdown 까진 진행한 뒤, Master 설정을 진행한다.
- 동일한 IP가 동일한 대역대에 살아있는 경우 네트워크 오류 발생

```shell
ifconfig
```
![Desktop View](/commons/MariaDB_MHA/01/24.png)
- 아직 IP가 지정되지 않은 ens34 인터페이스를 사용하여 VIP를 설정할 것이다.

```shell
vi /etc/sysconfig/network-scripts/ifcfg-ens34
```
- 위 네트워크 설정과 동일하게 변경해주되, IP는 VIP로 지정한 192.168.108.103으로 설정해준다.

```shell
service network restart
ifconfig
# IP가 성공적으로 설정되었는지 확인한다.
```
![Desktop View](/commons/MariaDB_MHA/01/25.png)
```shell
# Slave 서버
/sbin/ifdown ens34
ifconfig
# 성공적으로 해제되었는지 확인한다.
```
![Desktop View](/commons/MariaDB_MHA/01/26.png)

### Selinux Disabled 지정하기
```shell
vi /etc/selinux/config
    SELINUX=disabled
reboot
```

## 참고 URL
- https://m.blog.naver.com/jaechuns/221276414583
- https://mcc96.tistory.com/51