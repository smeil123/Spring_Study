

## Mysql DB와 spring boot 연동하기

RDB를 실제로 많이 사용하고, 디비구조나 sql사용법을 다시 공부하고 싶었기 때문에 mysql을 설치해서 사용하기로 했다.

**mysql 설치**
1. *https://dev.mysql.com/downloads/installer/* 여기서 mysql 서버와 shell 설치
2. 환경변수 path에 C:\Program Files\MySQL\MySQL Server 8.0\bin 추가


**spring boot와 연동하기**

1. build.gradle 에 의존성 추가
```
compile('mysql:mysql-connector-java')
compile('org.springframework.boot:spring-boot-starter-data-jpa')
```
2. application.properties


## 로그인 세션 정보를 Mysql 과 연동하기

1. build.gradle 에 의존성 추가
```
compile('org.springframework.session:spring-session-jdbc')
```

2. application.properties

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4OTUyMzgxOTEsMTg5NDcxNTEwN119
-->