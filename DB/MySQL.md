# MySQL 

* 가끔씩 사용하는 명령어들 모음

## 특정 테이블만 덤프뜨기
```sh
$   -u <user-name> -p <database> [<table-name> ...] > dump-file-name.sql
```
* 간혹 아래같은 에러가 뜨기도 하는데 `--single-transaction` 옵션을 사용하면 된다.
```
mysqldump: Got error: 1044: Access denied for user 'myuserid'@'%' to database 'mydatabasename' when doing LOCK TABLES
```
* `--single-transaction` : 덤프 도중 다른 세션에서 insert, update, delete 가능

## .sql 덤프 파일 로드하기
```sh
$ mysql -h <host> -u <user-name> -p db2 < dump-file-name.sql
```

## json, csv 포맷으로 테이블 내보내기
* 여러가지 방법이 있겠지만 MySQL Workbench 의 data export 기능을 쓰는 것이 편리하다

## 초기 비밀번호 세팅
- mysql -uroot
- ALTER USER 'root'@'localhost' IDENTIFIED BY '새로운 비밀번호';
- create database bboard

## 외부 접속권한 주기
  - grant all on bboard.* to root@'211.56.96.51' identified by '비밀번호';
  - grant all on bboard.* to root@'133.186.%.%' identified by '비밀번호';
## utf 관련 (database 만들자마자 설정해주기)
- ALTER SCHEMA database_name DEFAULT CHARACTER SET utf8mb4  DEFAULT COLLATE utf8mb4_unicode_ci ;
- ALTER TABLE tablename CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

## DB 스키마 확인
- select * from information_schema.SCHEMATA where schema_name=database_name AND table_name=table_name;
