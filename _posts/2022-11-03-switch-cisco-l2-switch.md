---
title: Switch_Cisco L2 Switch
author: B
date: 2022-11-03 11:03:00 +0900
categories: [System]
---

## VLAN 관련 메모사항
```
sw# show vlan
sw# conf t
sw(config)# vlan2
```

## ARP 관련 메모 사항
- ARP 테이블 내에서 포트 추적 확인하기
- show arp는 특정 interface에 직접적으로 들어가서 확인이 가능
    + sw(config)# show arp access-lists 명령어 사용
        * 현재 스위치 설치 환경으로 인한 결과 도출이 없음

```
sw# show ip arp [IP address]
```

- 위 명령어로 IP ARP Table 확인이 가능
- Total number of entries가 0일 경우에는 ARP Table이 등록되지 않았음을 뜻함. 즉, ARP Table은 Gateway 환경을 거쳐가면서 등록이 되는데, 현재 테스트하는 환경은 Gateway를 구축하지 않고 L2 스위치 간의 통신만을 지원하면서 ARP Table 결과가 없을 수 있다.

## 참고 URL
<https://jsson.tistory.com/25>