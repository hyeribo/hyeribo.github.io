---
title: "[AWS] RDS로 MariaDB 데이터베이스 생성하기"
date: 2019-02-16 20:57:00 +0900
categories: server
tags: aws rds mariadb
---

지금까지 클라이언트용 앱 / 웹은 많이 만들어보았다.  
하지만 api / 데이터베이스 서버는 로컬로만 띄워봤지, 실제 프로덕션용으로 배포하고 운영해본적은 없었다.  
오늘은 데이터베이스 서버를 AWS의 RDS를 이용해서 쉽게 만들어보자.  
데이터베이스 엔진에는 AWS에서 제공하는 AuroraDB도 있지만 나는 친숙한 MariaDB를 사용하겠다.

<br/>

## RDS console 접속
당연히 RDS를 사용하려면 AWS 계정이 필요하다. 로그인 후, [RDS console](https://console.aws.amazon.com/rds/)로 이동한다.

<br/>

## 인스턴스 생성
**Amason RDS > 데이터베이스** 메뉴로 이동, `데이터베이스 생성` 버튼 클릭
<br/>

![이미지](/assets/images/posts/20190216_1.png)

<br/>

**엔진선택** - `MariaDB` 선택, 하단 `RDS 프리티어에 적용되는 옵션만 사용` 체크박스에 체크하고 `다음 단계` 버튼 클릭
<br/>

![이미지](/assets/images/posts/20190216_2.png)

 <br/>
 
**DB 세부 정보 지정** - 인스턴스 사양 항목 전부 default
<br/>

![이미지](/assets/images/posts/20190216_3.png)

<br/>

**DB 세부 정보 지정** - 설정 항목 세팅 후, `다음 단계` 버튼 클릭
  - db 인스턴스 식별자 : 인스턴스 이름 지정
  - 마스터 사용자 이름 : mariadb 접속시 사용하는 마스터계정 아이디
  - 마스터 암호 : mariadb 접속시 사용하는 마스터계정 비밀번호
<br/>

![이미지](/assets/images/posts/20190216_4.png)

<br/>
 
**고급 설정 구성** - 네트워크 및 보안 항목 세팅
, 하위 항목 전부 default로 세팅 후, `인스턴스 생성` 버튼 클릭
  - VPC : 기본 VPC
  - 서브넷 그룹 : default
  - 퍼블릭 액세스 가능성 : 예
  - 가용영역 : 기본 설정 없음
  - VPC 보안그룹 : 기존 VPC 보안그룹 사용 - default 선택
<br/>

![이미지](/assets/images/posts/20190216_5.png)

<br/>

**고급 설정 구성** - 데이터베이스 옵션 항목 세팅
- 데이터베이스 이름 : demodb (기본으로 생성할 데이터베이스 이름)
- 포트 : 3306
- DB 파라미터 그룹 : 기본 선택값
- 옵션 그룹 : 기본 선택값
<br/>

![이미지](/assets/images/posts/20190216_6.png)
<br/>

하위 항목들(암호화, 백업, 모니터링, 로그 내보내기, 유지관리, 삭제방지)은 전부 default 세팅후, `데이터베이스 생성` 버튼 클릭


## DB 인스턴스에 연결
인스턴스 생성이 완료되었으면 데이터베이스에 연결해보자. 나는 맥용 데이터베이스 GUI 툴인 sequel pro를 사용했다.
sequel pro 앱을 열고 인스턴스 정보를 입력후, `Connect` 버튼을 클릭한다.
- Host: 인스턴스의 endpoint
- Username: 인스턴스 생성시 입력했던 `마스터 사용자 이름`
- Password: 인스턴스 생성시 입력했던 `마스터 암호`
<br/>

![이미지](/assets/images/posts/20190216_7.png)

### 만약 연결에 실패한다면 보안그룹을 확인하자.
1. 인스턴스 상세정보의 **VPC 보안 그룹** 항목의 보안그룹을 클릭해 보안그룹 페이지를 연다.
2. 하단 인바운드 탭의 `편집` 버튼을 클릭한다.
3. `규칙 추가` 버튼을 클릭해 규칙을 추가하고 저장한다.
  - 유형: MYSQL/Aurora
  - 소스: 내 IP

<br/>

연결후 Choose Database... 를 눌러보면 셀렉트박스 하단에 인스턴스 생성시 함께 생성한 `demodb` 데이터베이스가 보일것이다. 연결 성공!