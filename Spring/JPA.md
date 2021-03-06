
# JPA 란?

관계형 데이터베이스는 SQL을 통해서만 데이터를 접근할 수 있다.
그렇기에 프로젝트를 진행하면 코드의 대부분은 SQL로 작성되어진다.

### 문제점
RDB는 어떻게 데이터를 저장할지에 초점이 맞춰진 기술이나,
객체지향 프로그래밍 언어인 JAVA는 메세지를 기반으로 기능과 속성을 한 곳에서 관리하는 기술이다.

둘의 시작점이 다르기 때문에, 서로를 이해하는데 어려움이 존재했고 이를 완하하고자 JPA라는 기술을 사용한다.

즉, JPA 를 이용해 SQL에 종속적인 개발을 하지 않고, 객체지향적인 프로그래밍을 할 수 있게 한다.

#### 예시
```
User user = findUser();
Group group = user.getGroup();
```
이 코드를 보면, User-Group = 부모-자식 관계임을 알 수 있다.

반대로, 아래의 코드는 DB조회를 위한 코드이다
```
User user = userDao.findUser();
Group group = groupDao.findGroup(user.getGroupId());
```
user의 그룹아이디로 group정보를 받아오는 건 알 수 있으나, 둘 사이가 어떤 관계인지는 알 수 없다.


### Spring Data JPA
JPA는 인터페이스로서 자바 표준명세서이다.
* 인터페이스인 JPA를 사용하기 위해서는 구현체가 필요하다.
	* 대표적으로 Hibernate, EclipseLine 등
* 하지만 Spring에서는 이 구현체들을 직접 다루진 않고, Sping Data JPA사용
	* JPA <- Hibernate <- Spring Data JPA

왜 한단계 더 감싸둔 Spring Data JPA를 사용하는가?
* 구현체 교체의 용이성
	* Hibernate외에 다른 구현체로 쉽게 교체하기 위함
* 저장소 교체의 용이성
	* RDB외에 다른 저장소로 쉽게 교체하기 위함


### Entity 클래스와 기본 Entity Repository를 생성해서 사용한다

* 두 개 파일은 같은 경로에 존재해야 한다
* 기본 Repository 없이는 Entity클래스의 역할을 할 수 가 없다.
* 
예시 entity 클래스
```java
package com.springboot.project.domain.posts;  
  
import lombok.Builder;  
import lombok.Getter;  
import lombok.NoArgsConstructor;  
  
import javax.persistence.Column;  
import javax.persistence.Entity;  
import javax.persistence.GeneratedValue;  
import javax.persistence.GenerationType;  
import javax.persistence.Id;  
  
@Getter  
@NoArgsConstructor // 기본 생성자 자동 추가, public Posts() {}와 같은 효과  
@Entity  
public class Posts {  
  
    @Id // 해당 테이블의 PK필드를 나타냄  
  @GeneratedValue(strategy = GenerationType.IDENTITY) //PK생성규칙  
  private Long id;  
  
  // 테이블의 컬럼, 굳이 안써도되지만 기본값 외에 추가로 변경이 필요한 옵션이 있으면 사용  
  @Column(length = 500, nullable = false)  
    private String title;  
  
  @Column(columnDefinition = "TEXT", nullable = false)  
    private String content;  
  
 private String author;  
  
  @Builder // 해당 클래스의 빌더 패턴 클래스 생성, 생성자 상단에 선언 시 생성자에 포함된 필드만 빌더  
  // 생성자나 빌더나 생성 시점에 값을 채워주는 역할은 똑같으나,  
 // 생성자의 경우 지금 채워야할 필드가 무엇인지 지정할 수가 없다  
  public Posts(String title, String content, String author){  
        this.title = title;  
 this.content = content;  
 this.author = author;  
  }  
  
}
```

entity repository
```java
package com.springboot.project.domain.posts;  
  
import org.springframework.data.jpa.repository.JpaRepository;  
  
public interface PostsRepository extends JpaRepository<Posts, Long>{  
  
}
```

### JPA 테스트 코드

1. 테스트용 데이터 insert
2. 데이터 조회하여 값 비교

```java
import org.junit.After;  
import org.junit.Test;  
import org.junit.runner.RunWith;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.boot.test.context.SpringBootTest;  
import org.springframework.test.context.junit4.SpringRunner;  
  
import java.util.List;  
  
import static org.assertj.core.api.Assertions.assertThat;  
  
@RunWith(SpringRunner.class)  
@SpringBootTest  
public class PostsRepositoryTest {  
    @Autowired  
    PostsRepository postsRepository;  
  
  @After // Junit에서 단위 테스트가 끝날 때마다 수행되는 메소드 지정  
  // 보통은 배포 전 전체 테스트를 수행할 때 테스트간 데이터 침범을 막기 위함  
  // ex) 여러 테스트를 동시에 수행하면, 테스트용 데이터가 남아있어 영향을 줄 수 있음  
  public void cleanup(){  
        postsRepository.deleteAll();  
  }  
  
    @Test  
    public void 게시글저장_불러오기(){  
        //given  
  String title = "테스트게시글";  
  String content = "테스트본문";  
  
  // posts테이블에 insert/update 실행  
  // id가 있으면 update, 없으면 insert  postsRepository.save(Posts.builder()  
                .title(title)  
                .content(content)  
                .author("eunji@gmail.com")  
                .build());  
  
  //when  
  List<Posts> postsList = postsRepository.findAll(); // posts테이블에 있는 모든 데이터 조회  
  
  //then  
  Posts posts = postsList.get(0);  
  assertThat(posts.getTitle()).isEqualTo(title);  
  assertThat(posts.getContent()).isEqualTo(content);  
  }  
}
```

### JPA실행결과 확인방법

/src/main/resources 경로에 application.properties 파일을 생성하고 아래 코드를 추가
![jpa실행결과_설정](https://github.com/smeil123/CS_Study/blob/master/image/jpa실행결과_설정.PNG)

mysql 쿼리문 형식으로 결과를 조회하고 싶으면, 아래 코드 한 줄 더 추가
```
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
```


하고, JPA테스트 결과를 실행하면 아래와 같이 JPA로 실행된 SQL을 볼 수 있다
![jpa실행결과](https://github.com/smeil123/CS_Study/blob/master/image/jpa실행결과.PNG)



## dirty checking(더티 체킹)

Posts.java
```java
public void update(String title, String content){  
  this.title = title;  
    this.content = content;  
}
```

update를 할때 쿼리를 날리지 않아도 된다. **JPA의 영속성 컨텍스트** 때문에 가능하다.

#### 영속성 컨텍스트 
 * 엔티티를 영구 저장하는 환경
* 일종의 논리적인 개념
* JPA의 엔티티 매니저가 활성화된 상태로(Spring Data JPA의 기본옵션) 트랜잭션 안에서 데이터베이스에서 데이터를 가져오면 이 데이터는 영속성 컨텍스트가 유지된 상태

영속성 컨텍스트가 유지된 상태에서 데이터의 값을 변경하면 트랜잭션이 끝나는 시점에 해당 테이블에 변경분을 반영한다.
즉, Entity 객체의 값만 변경하면 별도로 Update쿼리를 날릴필요가 없어지며 이 개념을 **dritey checking**이라 한다.


## JPA Auditing으로 생성시간/수정시간 자동화하기

domain/BaseTimeEntity 생성
```java
package com.springboot.project.domain;  
  
import lombok.Getter;  
import org.springframework.data.annotation.CreatedDate;  
import org.springframework.data.annotation.LastModifiedBy;  
import org.springframework.data.annotation.LastModifiedDate;  
import org.springframework.data.jpa.domain.support.AuditingEntityListener;  
  
import javax.persistence.EntityListeners;  
import javax.persistence.MappedSuperclass;  
import java.time.LocalDateTime;  
  
@Getter  
@MappedSuperclass // JPA Entity클래스들이 BaseTimeEntity를 상속할 경우 필드들도 컬럼으로 인식하도록  
@EntityListeners(AuditingEntityListener.class) //BaseTimeEntity클래스에 Auditing기능을 포함  
public class BaseTimeEntity {  
  @CreatedDate // Entity가 생성되어 저장될 때 시간이 자동 저장  
  private LocalDateTime createdDate;  
  
    @LastModifiedDate // 조회한 Entity 의 값을 변경할 때 시간이 자동 저장  
  private LocalDateTime modifiedDate;  
  
}
```

이렇게 한 뒤, Posts 클래스(Entity 클래스)가 BaseTimeEntity를 상속받도록 변경
```java
public class Posts extends BaseTimeEntity {
```

마지막으로 JPA Auditing 어노테이션들을 모두 활성화할 수 있도록 application 클래스에 활성화 어노테이션 추가

```java
package com.springboot.project;  
  
import org.springframework.boot.SpringApplication;  
import org.springframework.boot.autoconfigure.SpringBootApplication;  
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;  
  
@EnableJpaAuditing // JPA Auditing 활성화  
@SpringBootApplication  
public class Application {  
  public static void main(String[] args) {  
  SpringApplication.run(Application.class, args);  
    }  
}
```

## Save 동작 원리

* entity 가 새로 생성될 예정 -> persist() 호출
* entity 가 기존에 있음 -> merge() 호출

### 새로 생성될지 아닐지 판단 기준
Entity가 언제 new로 인식되는지

**식별자(@Id)**
* 식별자가 null 또는 0 인 경우
* 이때, Long타입과 같이 Wrapper일 경우 null을 newState로 인식하고, int와같이 Primitive일 경우 new상태로 인식합니다 (무슨말인지 잘 모르겠음)

**@Version**
* Entity 필드에 있는 @Version이 null 인 경우
* 이때, @Verion 값이 null + @Id에 값이 존재 -> newState
> @Version : Entity에서 Lock을 잡고 싶을 때 사용 ??

**@Persistable**
* Persistable interface를 구현한 경우 => 개발자가 정의하는 대로
	* Persistable interface 는 getId(), isNew() 메소드를 제공하는 인터페이스
	* 즉, Persistable을 구현한 Entity는 구현한 isNew() 메소드에 따라 new인지 아닌지 판단
* 이때는 @Id, @Version 은 new를 구별하는 기준이 되지 않는다.

**Entity Information customize**
* EntityInformation을 customize하는 경우, 위와 비슷하게 개발자가 커스터마이증한대로 isNew 를 판단

### Persist
* 입력받은 데이터를 가지고  insert 문을 바로 호출

### Merge
* 데이터를 만들 때 key값을 알고, key를 기준으로 select해서 없으면 insert 문 호출
* select -> insert 단계로 수행되기 때문에 데이터가 많으면 **성능 저하**를 일으킴

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzNDg2MzAwODIsLTIwMDQ1MjY2NzksLT
QwMjE4NzAxNCwxOTg5NDMyMTA1LC0xNTAwMzI4MTk4LDE2OTMw
ODYyMzQsLTIwODAwNTgxMDMsMTM4MTkxNTc5Ml19
-->