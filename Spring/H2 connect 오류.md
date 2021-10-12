

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
하지만 h2-console에서는 CSRF처리가 되어 있지 않아 방지장치 해제를 해줘야 정상접근이 가능하다. 전체를 disable하는 방법도 있고, h2-console에서만 예외하는 방법이 있는데 후자를 적용했다.
```java
.csrf()  
  .ignoringAntMatchers("/h2-console/**")  
.and()
```

### 3. X-Frame-Options 예외 처리
이 옵션은 HTTP응답 헤더의 fram, iframe, object에 렌더링할 수 있는지 여부인데, 사이트 내 콘텐츠들이 다른 사이트에 포함되

이 부분을 설정해주지 않으면 화면 접근은 가능하되, 다 깨져서 제대로 보이지 않는다.
```java
.headers().addHeaderWriter(new XFrameOptionsHeaderWriter(XFrameOptionsHeaderWriter.XFrameOptionsMode.SAMEORIGIN))  
.and()
```


**전체코드**
```java
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
import org.springframework.security.config.annotation.web.builders.HttpSecurity;  
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;  
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;  
import org.springframework.security.crypto.factory.PasswordEncoderFactories;  
import org.springframework.security.crypto.password.PasswordEncoder;  
import org.springframework.security.web.header.writers.frameoptions.XFrameOptionsHeaderWriter;  
  
@Configuration  
@EnableWebSecurity  
public class SpringSecurityConfig extends WebSecurityConfigurerAdapter {  
  @Override  
  protected void configure(HttpSecurity http) throws Exception{  
 http  .authorizeRequests()  
  .antMatchers("/h2-console/**").permitAll()  
  .and()  
  .csrf()  
  .ignoringAntMatchers("/h2-console/**")  
  .and()  
  .headers().addHeaderWriter(new XFrameOptionsHeaderWriter(XFrameOptionsHeaderWriter.XFrameOptionsMode.SAMEORIGIN))  
  .and()  
  .formLogin()  
  .loginPage("/login")  
  .permitAll()  
  .and()  
  .logout();  
    }  
  
  @Bean  
  public PasswordEncoder passwordEncoder(){  
  return PasswordEncoderFactories.createDelegatingPasswordEncoder();  
    }  
}
```

**참고**
*https://github.com/HomoEfficio/dev-tips/blob/master/Spring%20Security%EC%99%80%20h2-console%20%ED%95%A8%EA%BB%98%20%EC%93%B0%EA%B8%B0.md*
<!--stackedit_data:
eyJoaXN0b3J5IjpbNTExNzIwNTAyXX0=
-->