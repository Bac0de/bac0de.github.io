---
layout: post
title:  "로컬네트워크의 통신 설계"
date:   2017-09-22 09:41:09
author: SeokHo Song
---



유저레벨에서 가상화 기술을 손쉽고 가볍게 사용 할 수있게 해줄 Batu 프로젝트에서 각각의 VM을 설정하거나, Local Area Network (LAN) 에서 각각의 Batu를 설정 할 수 있게 해줄 Control Panel을 개발하게 되었다.

Linux Kernel 기반의 GUI 배포판인 Ubuntu 16.04 LTS로 테스트 및 개발하고 있다.

본 프로젝트를 기획하면서 GUI Library 사용에 대해 고민을 했다.

처음 기획으론 QT를 사용하려 했으나, Cross-Platform 및 Web에 대한 호환성에 강점이 있는 Electron을 사용하기로 결정했다.



Batu의 Control Panel은 각각 정의되어있는 VM에 대한 수정과, Local Network 상에 있는 Batu의 VM 제어가 가능하도록 설계하였다.

UI FrameWork ( ex)Bootstrap, Materialize, etc...) 에 종속적이지 않기 위해 batu.js에서 모듈화를 진행해 개발하고 있다.


batu.js에서 제공하는 Http Server 기능을 사용하고자 한다면 다음과 같은 코드를 작성한다.

```javascript
	var batu = require('location batu.js');
	batu.Http_ServerStart(function(){});
```


Batu의 암호는 pwd/pwd 파일에 SHA512 알고리즘으로 암호화 된 문자열로 저장되어 있다.

Batu에서 암호는 두가지 역할을 한다. 

1. Batu Control Panel 접근용
	Batu는 컴퓨터의 BIOS SETUP 페이지에 접근과 유사하게 동작한다. BatuOS위에서 VM을 컬때 특정한 키를 입력하여 Control Panel에 접근할 수 있는데, 이때 사용자에 대한 접근 권한을 구분한다.

2. LAN Network의 Batu 제어 Http API에서 보안성 향상
	Batu는 내부적으로 text 정보교환을 자주 한다. 따라서, HTTP서버를 내부적으로 운영하는데, 이때 Hash화 한 암호를 Request Body 또는 Parameter에 제공하여, API사용 권한에 대한 인증을 진행한다.


1번에 대한 동작 과정은 다음과 같다.
	1. 사용자는 Batu의 VM을 설정하기 위해 BatuVM이 켜지는 과정에서 특정한 키( ex)F12 )를 입력한다.
	2. 사용자는 Control Panel에 접근하기 위해 암호를 입력한다.
	3. 암호에 대한 인증을 진행하고, 만약 올바르지 않은 암호가 3번 입력될 경우 1번으로 돌아간다.
	4. 만약 암호에 대한 인증이 성공하였다면, 사용자에게 Control Panel을 제공한다.

2번에 대한 동작 과정은 다음과 같다.
	1. 사용자의 행동 및 Program 의 필요에 의해 Local Network의 다른 Batu 시스템에 대한 특정한 요청을 요구한다.
	2. 요청을 보내는 쪽에서는 사용자가 로그인을 진행할 당시의 Hash Vaule를 Http Request에 실어서 보낸다.
	3. 요청을 받는 쪽에선, 보내는 쪽의 요청을 처리하기전에 요청하는 쪽의 정상적인 요청인지 판단하기 위해 보내는쪽에서 같이 전송한 Hash Value를 받는 쪽에서 보내는쪽으로 HTTP POST: /isAuth  요청을 보낸다. 요청 결과가 success라면 요청 처리를 한다.



2번 과정에서 보안적으로 취약한 점이 몇가지 보인다, 추후 개선 가능할 것으로 예상된다.



2017-10-18 updated

( 위에서 설계했던 내용은 현제 많은 부분 수정/삭제/추가 되었다. 본 내용과 비교는 무의미 할 것 같으므로  추후 재 포스팅으로 업데이트 할 것이다. ) 
