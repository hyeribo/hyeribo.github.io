---
title: "[MariaDB] Homebrew로 MariaDB(MySQL) 설치하기"
date: 2019-02-17 18:22:00 +0900
categories: server
tags: mariadb mysql
---

## MariaDB 설치
```
$ brew install mariadb
```

## MariaDB 서버 시작
```
$ mysql.server start
```

## 기본 설정
```
$ mysql_secure_installation
```
위의 명령어를 입력하고, 아래 질문사항에 대답한다.  

**Would you like to setup VALIDATE PASSWORD plugin?**
 : 복잡하고 안전한 비밀번호 플러그인을 사용하겠는가? (y/n)

**Remove anonymous users?**
 : 익명유저를 삭제하겠는가? (y/n)
 : n 입력시 명령어 ``mysql`` 만으로도 접속이 가능하다.

**Disallow root login remotely?**
 : root 계정 원격접속을 비허용하겠는가? (y/n)
 : y 입력시 root 계정으로 원격접속 불가능

**Remove test database and access to it?**
 : test 데이터베이스 삭제하겠는가? (y/n)

**Reload privilege tables now?**
 : privileges 테이블을 재시작 할 것인가? (y/n)
 : y를 입력하자.


## 사용자 추가
```
$ mysql -u root -p  // root 계정으로 로그인

$ create user '사용자 아이디'@'localhost' identified by '비밀번호'; // 로컬에서 접속가능한 유저 생성
$ create user '사용자 아이디'@'원격접속 IP' identified by '비밀번호'; // 해당 IP에서 접속가능한 유저 생성
$ create user '사용자 아이디'@'%' identified by '비밀번호';         // 모든 IP에서 접속가능한 유저 생성
```


## 데이터베이스 접속
나는 맥북을 사용하므로 sequel pro로 접속할것이다. 아래의 정보를 입력하고 Connect 버튼을 누른다.
- name: localhost 또는 원하는 이름 입력
- Host: 127.0.0.1
- Username: root 또는 위에서 추가했던 '사용자 아이디' 입력
- Password: 비밀번호 입력

![이미지](/assets/images/posts/20190217_1.png)

### 연결도중 에러가 발생한다면
![이미지](/assets/images/posts/20190217_2.png)
**Authentication plugin 'caching_sha2_password' cannot be loaded** 혹은 <br> **this authentication plugin is not supported** 에러가 발생한다면 아래의 명령어 실행
```
ALTER USER 'username'@'localhost' IDENTIFIED WITH mysql_native_password BY '비밀번호';
```
'username'@'localhost' 부분은 로그인할 유저 정보를 따른다.  
자세한 사항은 [이곳](https://ondemand.tistory.com/246)을 참고

## 기타 명령어
```
$ mysql.server stop    // 서버 중지
$ mysql.server restart // 서버 재시작
```