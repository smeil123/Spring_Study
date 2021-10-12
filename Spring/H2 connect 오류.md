

## spring boot 에서 H2 console Whitelabel Error Page오류

처음엔 원인을 모르고 설정이나 코드에 오류가 있는줄알고 한참헤맸는데,,
알고보니 spring security와 h2 database를 같이 쓰면 접속이 제대로 되지 않는 문제였다.

spring security를 적용하면 로그인을 해야 페이지에 접속할 수 있게되면서 h2console이 디폴트 로그인창으로 자동 리다이렉트 된다.
> 이때는 spring security를 제대로 적용해둔 상태는 아니여서 리다이렉트는 되지 않고 계속 오류가 났었는데, 그 부분 경로(/config)를 잠깐 다른곳에 옮겨두고 실행했더니 login 화면으로 계속 리다이렉트가 되서 알아챘다.

### 1. 인증 예외 처리
Spring Security에서 h2-console 경로를 예외로 추가
```java
.authorizeRequests()  
  .antMatchers("/h2-console/**").permitAll()
```

### 2. CSRF 예외 처리
Spring Security에는 Cross Site Request Forgery 방지 장치가 디폴트 설정이다.
하지만 h2-con

**참고**
*https://github.com/HomoEfficio/dev-tips/blob/master/Spring%20Security%EC%99%80%20h2-console%20%ED%95%A8%EA%BB%98%20%EC%93%B0%EA%B8%B0.md*
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE2NTA1MzQwNV19
-->