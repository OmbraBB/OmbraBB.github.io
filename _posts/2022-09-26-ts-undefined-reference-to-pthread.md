---
title: T/S_undefined reference to `pthread_...`
author: B
date: 2022-09-26 10:00:00 +0900
categories: [TroubleShoot]
image:
    path: /commons/20220926/01.png
---


![Desktop View](/commons/20220926/01.png)

해결방법 : 컴파일 시 gcc옵션에서 -lpthread 링크 포함

예시 : $ gcc -o test ./test.c -lpthread