

## spring boot 에서 H2 console Whitelabel Error Page오류

처음엔 원인을 모르고 설정이나 코드에 오류가 있는줄알고 한참헤맸는데,,
알고보니 spring security와 h2 database를 같이 쓰면 접속이 제대로 되지 않는 문제였다.

spring security를 적용하면 로그인을 해야 페이지에 접속할 수 있게되면서 h2console


**참고**
*https://github.com/HomoEfficio/dev-tips/blob/master/Spring%20Security%EC%99%80%20h2-console%20%ED%95%A8%EA%BB%98%20%EC%93%B0%EA%B8%B0.md*
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTkzNTUwNjg5XX0=
-->