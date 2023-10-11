---
title: T/S_RequestsDependencyWarning Message
author: B
date: 2023-04-18 14:11:00 +0900
categories: [TroubleShoot]
image:
    path: /commons/20230418/01.png
---

오랜만에 리눅스 컨테이너에 솔루션 서버 환경을 설치하는 도중에 아래와 같은 에러가 났다.

![Desktop View](/commons/20230418/01.png){: width="100%"}

패키지 설치 시, requests와 chardet 라이브러리를 설치하는데 둘 다 최신버전으로 자동 설치되어 생긴 두 라이브러리 간의 호환성 문제이다.

> 사진 상에서 chardet 버전은 오늘 날짜 기준 가장 최신 버전인 5.0.0이였다.
{: .prompt-info}

1. chardet을 2.1.1로 다운그레이드 했지만 동일한 경고문이 표출되었다.
2. chardet을 3.0.4로 다운그레이드 하니 경고문이 표출되지 않았다.
최종적으로 urllib3의 버전은 최신버전인 1.26.15를 그대로 사용하고 chardet의 버전을 3.0.4로 다운그레이드 해주었다.

다운그레이드 command는 아래와 같다.

```bash
pip install --upgrade chardet==3.0.4 --use-feature=2020-resolver
```

--use부분은 pip가 dependency 충돌 문제를 해결하는 방법이므로 적어주지 않으면 에러가 발생한다.

## 참고 URL
<https://ieworld.tistory.com/16>