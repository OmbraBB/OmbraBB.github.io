---
title: T/S_IPv6 duplicate Error
author: B
date: 2025-09-05 17:31:00 +0900
categories: [TroubleShoot]
---

- 개요 :
    - 가상 머신 내, Rocky linux 9.5를 설치한 후 Rocky 기본 네트워크 설정인 Network Manager 환경을 CentOS와 같이 network-scripts 환경으로 변경한 뒤 네트워크 재시작을 하니 "IPv6: ensXXX: IPv6 duplicate address ... used by ... detected! " 와 같은 에러가 발생하였다.

- 해결 방안 :
    - 위 에러는 Network Manager 서비스와 network-scripts 서비스의 충돌로 인해 발생했을 가능성이 크다.
    - 그 경우, network0scripts 에서 IPV6INIT와 IPV6_AUTOCONF를 no로 변경한 뒤 Network Manager에서 ipv6메소드를 disabled 해준다.

    ```shell
    nmcli connection modift endXXX piv6.method "disabled"
    nmcli dev reapply ensXXX
    ```

    - 재시작시에도 Network Manager가 재동작 하지 않도록 해당 서비스에 mask 처리를 한다.

    ```shell
    systemctl stop NetworkManager
    systemctl disable NetworkManager
    systemctl mask NetworkManager
    ``` 