---
title: VScode_Git_VScode와 Git 연동
author: B
date: 2022-09-19 17:58:00 +0900
categories: [VScode, Git]
---

![Desktop View](/commons/20220919/01.png){: width="477" height="381" .w-75 .normal}

- `Download Git for windows` 버튼 클릭

![Desktop View](/commons/20220919/02.png){: width="477" height="381" .w-75 .normal}

- `Open` 버튼 클릭

![Desktop View](/commons/20220919/03.png){: width="477" height="381" .w-75 .normal}

- `64-bit Git for Windows Setup.` 버튼 클릭 후 아래에 뜨는 셋업 프로그램 클릭

<p width="100%">
    <a href="/commons/20220919/04.png" class=""><img src="/commons/20220919/04.png" width="344" height="270" ></a>
    <a href="/commons/20220919/05.png" class=""><img src="/commons/20220919/05.png" width="344" height="270" ></a>
</p>

- Install 버튼이 나올 때 까지 Next 버튼을 클릭한다.
- Install 버튼이 나올 때는 선택사항 클릭 후 Install 버튼을 클릭한다.

<p width="100%">
    <a href="/commons/20220919/06.png" class=""><img src="/commons/20220919/06.png" width="344" height="270"></a>
    <a href="/commons/20220919/07.png" class=""><img src="/commons/20220919/07.png" width="344" height="270"></a>
</p>

- git 다운로드를 진행하며, 진행 후 Finish 버튼을 클릭한다.

![Desktop View](/commons/20220919/08.png){: width="477" height="381" .w-75 .normal}

- 설치 완료 후 `reload`버튼 클릭

<p width="100%">
    <a href="/commons/20220919/09.png" class=""><img src="/commons/20220919/09.png" width="344" height="270"></a>
    <a href="/commons/20220919/10.png" class=""><img src="/commons/20220919/10.png" width="344" height="270"></a>
</p>

- `Clone Git Repository` 클릭 후 git clone 주소 복사

![Desktop View](/commons/20220919/11.png){: width="477" height="381" .w-75 .normal}

- 위와 같이 창에 주소를 복사해준다.

![Desktop View](/commons/20220919/12.png){: width="477" height="381" .w-75 .normal}
<p width="100%">
    <a href="/commons/20220919/13.png" class=""><img src="/commons/20220919/13.png" width="480" height="143"></a>
    <a href="/commons/20220919/14.png" class=""><img src="/commons/20220919/14.png" width="207" height="143"></a>
</p>

- 데스크탑 내 Git과 연동할 폴더 선택 및 레포지토리 연동

<p width="100%">
    <a href="/commons/20220919/15.png" class=""><img src="/commons/20220919/15.png" width="243" height="240"></a>
    <a href="/commons/20220919/16.png" class=""><img src="/commons/20220919/16.png" width="444" height="240"></a>
</p>

- index.html 파일 생성 후 임의의 내용 작성

&nbsp;

- ctrl + ` 버튼 클릭 후 터미널 창에서 오른쪽 + 버튼 클릭
- git bash 버튼 클릭
- 아래 명령어 순차적 기입
    ```
    $ git config --global user.email "[Email 주소]"
    $ git add *
    $ git commit -m "Start git blog"
    $ git push
    # -> 인증 창이 열리면, 인증 후 깃허브 페이지 새로 고침 시 push 확인 가능
    ```