

## Mysql DB와 spring boot 연동하기

RDB를 실제로 많이 사용하고, 디비구조나 sql사용법을 다시 공부하고 싶었기 때문에 mysql을 설치해서 사용하기로 했다.

**mysql 설치**
1. *https://dev.mysql.com/downloads/installer/* 여기서 mysql 서버와 shell 설치
2. 환경변수 path에 C:\Program Files\MySQL\MySQL Server 8.0\bin 추가

**mysql 스키마 & 테이블 생성**
spring boot에 내장된 h2 데이터베이스를 사용할 땐 domain에 java로 필요한 컬럼 생성해주면 자동을 테이블이 만들어졌었는데, 이를 mysql에 직접 넣어주려다가 놓치는 부분이 생겨서 테이블 alter로 조정해줬다.
테이블을 추가하게되면 테이블 구조를 미리 생각해 둔 다음에 그려야겠다.
```
CREATE DATABASE post_db default CHARACTER SET UTF8;  
```
기존에 생성되어 있던 db 테이블과 컬럼을 모두 만들어 준다
```
create table if not exists post_db.post(
    -> id int(10) NOT NULL,
    -> title varchar(100) NOT NULL,
    -> content TEXT(5000),
    -> author varchar(100) NOT NULL,
    -> read_count int(10));

alter table post_db.posts add column modified_date datetime;
alter table post_db.posts add column created_date datetime;

# 기본키값 설정도 해주기
alter table post_db.posts modify column id int(10) not null auto_increment primary key first;```


**spring boot와 연동하기**

1. build.gradle 에 의존성 추가
```
compile('mysql:mysql-connector-java')
compile('org.springframework.boot:spring-boot-starter-data-jpa')
```
2. application.properties
```
spring.datasource.url=jdbc:mysql://localhost:3306/post_db?useSSL=false&useUnicode=true&serverTimezone=Asia/Seoul  
spring.datasource.username=sysadmin  
spring.datasource.password=tlsgksdmsgod123!
```

## 로그인 세션 정보를 Mysql 과 연동하기

1. build.gradle 에 의존성 추가
```
compile('org.springframework.session:spring-session-jdbc')
```

2. application.properties

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQzNzE0NDE3MCwtMTU1NDI3OTM0NSw3OD
MyODUzMzgsLTQ5NjgxNzE0NSwxODk0NzE1MTA3XX0=
-->