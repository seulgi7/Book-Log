# CHAPTER01. 인텔리제이 스프링 부트 시작하기.
- 프로젝트 환경
  - Java : 8
  - Gradle : 4.10.2

# CHAPTER02. 스프링 부트에서 테스트 코드를 작성하자.

## 2.1 테스트 코드 소개

TDD와 단위 테스트(Unit Test)는 다른 이야기.

### TDD

- 테스트가 주도하는 개발.

- 테스트 코드를 먼저 작성하는 것부터 시작한다.

- 레드 그린 사이클

  - (Red) 항상 실패하는 테스트를 먼저 작성하고
  - (Green) 테스트가 통과하는 프로덕션 코드를 작성하고
  - (Refactor) 테스트가 통과하면 프로턱션 코드를 리팩토링 한다.

  <img src="https://blog.kakaocdn.net/dn/cECfwD/btqBUTp55pN/kpRuUmLg6k7IzmDr12aFS0/img.gif" alt="햄코미 Study Stack :: TDD(Test-Driven Development)란?" style="zoom:50%;" />

### 단위 테스트 (Unite Test)

- 단위 테스트란?

  - 기능 단위의 테스트 코드를 작성하는 것.

  - TDD와 달리 테스트 코드를 꼭 먼저 작성해야 하는 것도 아니고, 리팩토링도 포함되지 않는다.

  - 순수하게 테스트 코드만 작성하는 것을 이야기 함.

- 장점

  - 개발단계 초기에 문제를 발견하게 도와준다.

  - 개발자가 나중에 코드를 리팩토링하거나 라이브러리 업그레이드 등에서 기존 기능이 올바르게 작동하는지 확인할 수 있다.  ex)회귀 테스트.

  - 기능에 대한 불확실성을 감소시킬 수 있다.

  - 시스템에 대한 실제 문서를 제공한다. 즉, 단위테스트 자체가 문서로 사용할 수 있다.

    

- 필자의 경험담

  - 필자가 단위 테스트를 배우기 전 진행한 개발방식.

    1. 코드를 작성하고
    2. 프로그램(Tomcat)을 실행한 뒤
    3. Postman과 같은 API 테스트 도구로 HTTP 요청하고
    4. 요청결과를 System.out.println()으로 눈으로 검증한다.
    5. 결과가 다르면 다시 프로그램(Tomcat)을 중지하고 코드를 수정한다.

    이 때, 2~5는 매번 코드를 수정할 때마다 반복해야만 한다.

  - 단위테스트후 바뀐 변화

    - 톰캣을 계속 내렸다가 다시 실행하는 일의 반복이 없어짐.
      - 수정할때마다 톰캣을 재구동하며 확인하는 일은 테스트 코드가 없다 보니 눈과 손으로 직접 수정된 기능을 확인할 수 밖에 없기 때문이다.
    - System.out.println()을 통해 눈으로 검증해야하는 수동검증이 필요 없어짐.
      - 테스트 코드를 작성하면 더는 사람이 눈으로 검증하지 않게 자동검증이 가능하다.
    - 개발자가 만든 기능을 안전하게 보호해준다.
      - 새로운 기능이 추가될 때, 기존 기능이 잘 작동되는 것을 보장해 주는 것.



- 테스트 코드 작성을 도와주는 프레임 워크 
  - Junit - Java
  - DBUnit - DB

## 2.2 Hello Controller 테스트 코드 작성하기

>  패키지 네이밍 규칙 : 일반적으로 패키지명은 웹 사이트 주소의 역순으로 한다.

### @SpringBootApplication

- @SpringBootApplication으로 인해 스프링 부트의 자동 설정, 스프링 Bean 읽기와 생성을 모두 자동으로 설정된다.
- 위치 : 항상 **프로젝트 최상단에 위치**해야 한다.
  - 왜? @SpringBootApplication이 있는 위치 부터 설정을 읽어가기 때문이다.

```java
package com.spring.admin;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```



> 스프링 부트에서는 **내장  WAS** 를 사용하는 것을 권장한다.
>
> - 왜? **언제어디서나 같은 환경에서 스프링 부트를 배포**할 수 있기 때문이다.
>
>   - 내장 WAS를 사용하면 항상 서버에 톰캣을 설치할 필요가 없게되고, 스프링 부트로 만들어진  Jar 파일(실행 가능한 Java 패키징 파일)을 실행하면 됨. 
>     - 내장 WAS란 별도로 외부에  WAS를  두지 않고 애플리케이션을 실행 할 때 내부에서 WAS를 실행하는 것을 이야기함.
>   - 만약 외장 WAS를 사용한다고 하면 모든 서버는 WAS의 종류와 버전, 설정을 일치시켜야만 한다. 
>
> 



### @RequestController

- 컨트롤러를 JSON을 반환하는 컨트롤러로 만들어 준다.
- 예전에는 @ResponserBody를 각 메소드마다 선언했던 것을 한번에 사용할 수 있게 해준다고 생각하면 된다.

### @GetMapping

- HTTP Method인 Get의 요청을 받을 수 있는 API를 만들어준다.
- 예전에는 @RequestMapping(method = RequestMethod.GET)으로 사용되었다. 

### @RunWith(SpringRunner.class)

- 테스트를 진행할 때 JUnit에 내장된 실행자 외에 다른 실행자를 실행시킨다.
  - 여기서는 SpringRunner라는 스프링 실행자를 사용한다.
- 스프링 부트 테스트와 JUnit 사이에 연결자 역할을 한다.

### @ WebMvcTest

- 여러 스프링 테스트 어노테이션 중, Web(Spring MVC)에 집중할 수 있는 어노테이션.
- 선언할 경우 @Controller, @ControllerAdvice 등을 사용하 수 있음.
- 단, @Service, @Component, @Repository등은 사용할 수 없음.

### @Autowired

- 스프링이 관리하는 빈(Bean)을 주입 받습니다.

### @private MockMvc mvc

- 웹 API를 테스트할 때 사용한다.
- 스프링 MVC 테스트의 시작점.
- 이 클래스를 통해 HTTP GET, POST 등에 대한 API 테스트를 할 수 있다.



## 2.3 롬복 소개 및 설치하기

- 롬복 ( Lombok )

  - 자바 개발자들의 필수 라이브러리.

  - 자바 개발할 때 자주 사용하는 코드 Getter, Setter, 기본생성자, toString 등을 어노테이션으로 자동 생성해줍니다.

  - 프로젝트에 롬복 추가 방법

    1. build.gradle  에 다음의 코드를 추가한다.

       ```
       compile('org.projectlombok:lombok')
       ```

    2. build.gradle 새로고침 → 라이브러리를 내려받는다.

    3. Lombok Plugin 설치

    4. 롬복 설정 : Settings > Build > Compiler > Annotation Processors

       ☐ Enable annocation processing 체크

       >  ✔️ 단, 롬복은 프로젝트마다 설정해야한다.  
       >
       >  - build.gradle에 라이버리를 추가하는 것과 Enable annotaion processing을 체크하는 것은 프로젝트마다 진행해야 한다.
       >  - 플러그인 설치는 한 번만 하면 된다.



## 2.4 Hello Controller 코드를 롬복으로 전환하기

### @Getter

- 선언된 모든 필드의 get 메소드를 생성해 줍니다.



### @ResponseArgsConstructor

- 선언된 모든 final 필드가 포함된 생성자를 생성해 줍니다.
- final이 없는 필드는 생성자에 포함되지 않습니다.



### assertThat

- assertj라는 테스트 검증라이브러리의 검증 메소드.
- 검증하고 싶은 대상을 메소드 인자로 받는다.
- 메소드 체이닝이 지원되어 isEqualTo와 같이 메소드를 이어서 사용할 수 있다.

> Junit과 비교하여 assertj의 장점
>
> - CoreMatchers와 달리 추가적으로 라이브러리가 필요하지 않다.
>   - Junit의 assertThat을 쓰게 되면 is()와 같이 CoreMatchers 라이브러리가 필요하다.
> - 자동완성이 좀 더 확실하게 지원된다.
>   - IDE에서는 CoreMatchersd와 같은 Matcher 라이브러리의 자동완성 지원이 약하다.



### isEqualTo

- assertj의 동등 비교 메소드.
- assertThat에 있는 값과 isEqualTo의 값을 비교해서 같을 때만 성공.



### @RequestParam

- 외부에서 API로 넘긴 파라미터를 가져오는 어노테이션.
- 여기서는 외부에서 name (@RequestParam("name") ) 이란 이름으로 넘긴 파라미터를 메소트 파라미터 name (String name)에 저장하게 된다.

### @param

- API 테스트할 때 사용될 요청 파라미터를 설정한다.
- 단, 값은 String만 허용된다.
- 숫자/날짜 등의 데이터도 등록할 때는 문자열로 변경해야만 가능하다.

### jsonPath

- JSON 응답값을 필드별로 검증할 수 있는 메소드.

- '$'를 기준으로 필드명을 명시한다. ex) name → $.name , amount → $.amount

  

# CHAPTER03. 스프링 부트에서 JPA로 데이터베이스 다뤄보자.



> JPA :자바 표준 ORM (Object Relational Mapping)
>
> > Q) Mybatis, iBatis는 ORM인가? 
> >
> > A) ORM(객체를 매핑)이 아니라 SQL Mapper(쿼리를 매핑).



## 3.1 JPA 소개

- 등장 배경

  - CRUD SQL을 매번 생성해야하는 **단순 반복 작업의 문제**.

    - 왜 ?

      - 현대의 웹 애플리케이션에서 **객체를 관계형 데이터베이스에서 관리**하는 것이 무엇보다 중요. 

      - 관계형 데이터베이스(RDB, Relational Database)는 빠질 수 없는 요소이기 때문에 Oracle, MySQL, MSSQL 등을 쓰지 않는 웹 애플리케이션은 거의 없기 때문이다.

        ➣   관계형 데이터베이스가 SQL만 인식할 수 있기 때문에, SQL로만 가능하니 각 테이블마다 기본적인 CRUD (Create,  Read, Update, Delete) 을 매번 생성해야 함.

        ➣  애플리케이션 코드보다 **SQL**로 가득하게 됨.

  - **패러다임 불일치**

    - 관계형 데이터베이스와 객체지향 프로그래밍 언어의 패러다임이 서로 다른데, 객체를 데이터베이스에 저장하려고 하니 여러 문제가 발생한다. 이를  **패러다임 불일치**라고 한다.
      - **관계형 데이터베이스** :  **어떻게 데이터를 저장**할지에 초점이 맞춰진 기술.
      - **객체지향 프로그래밍 언어** : 메시지를 기반으로 **기능과 속성을 한 곳에서 관리**하는 기술.
    - **영향** : 웹애플리케이션 개발은 점점 **데이터베이스 모델링**에만 집중하게 됨.
      - Why ? 상속, 1:N 등 다양한 객체 모델링을 데이터베이스로는 구현할 수 없기 때문.

  

- JPA 등장

  - 서로 지향하는 바가 다른 2개 영역(객체지향 프로그래밍 언어와 관계형 데이터베이스)을 **중간에서 패러다임 일치**를 시켜주기 위한 기술.
  - 장점
    - 개발자는 **객체지향적으로 프로그래밍**을 하고, JPA가 이를 관계형 데이터베이스에 맞게 SQL을 대신 생성해서 실행함.
    - 개발자는 항상 객체지향적으로 코드를 표현할 수 있으니 더는 **SQL에 종속적인 개발을 하지 않아도 된다.**
    - 객체 중심으로 개발을 하게 되니 **생산성 향상**은 물론 **유지 보수하기가 편함.**
      - 이런 점 때문에 365일 24시간, 대규모 트래힉과 데이터를 가진 서비스에서 JPA는 점점 표준 기술로 자리 잡고 있다.

###  Sping Data JPA

- JPA 와 구현체

  - JPA ? 인터페이스로서 자바 표준 명세서.

  - **인터페이스인 JPA**를 사용하기 위해서는 **구현체가 필요함**. ex)  Hibernate, EclipseLink 등

  - **Spring에서 JPA**를 사용할 때는 이 **구현체들을 직접 다루진 않는다.**
  
    - 구현체들을 좀 더 쉽게 사용하고자 추상화시킨 **Spring Data JPA라는 모듈을 이용하여 JPA 기술을 다룬다.**

    - 이들의 관계를 보면 아래와 같다.

      > ​	JPA ← Hibernate ← Spring Data JPA

    - Hibernate를 쓰는 것과 Spring Data JPA를 쓰는 것 사이에는 큰 차이가 없다. 그럼에도 스프링 진영에서는 Spring Data JPA를 개발했고, 이를 권장하고 있다.

      

- Spring Data JPA가 등장한 이유 

  - **구현체 교체의 용이성**
    - Hibernate 외에 **다른 구현체로 쉽게 교체**하기 위함.
    - **Spring Data JPA 내부에서 구현체 매핑을 지원**해주기 때문에 **교체가 용이**하다.
      - Hibernate가 언젠간 수명을 다해서 새로운 JPA 구현체가 대세로 떠오를 때, Spring Data JPA를 쓰는 중이라면 아주 쉽게 교체할 수 있다.
  - **저장소 교체의 용이성**
    - 관계형 데이터베이스 외에 **다른 저장소로 쉽게 교체**하기 위함.
      - 교체가 필요하다면 Spring Data JPA에서 Spring Data MongoDB로 **의존성만 교체하면 된다.**
        - 어떻게 가능? 이는 Spring Data의 하위 프로젝트들은 기본적인 CRUD의 인터페이스가 같기 때문이다.



### 실무에서 JPA

- 실무에서 JPA를 사용하지 못하는 가장 큰 이유? 높은 러닝 커브
  - JPA를 잘 쓰려면 객체지향 프로그래밍과 관계형 데이터베이스 둘 다 이해해야 함.
- JPA를 사용해서 얻는 보상
  - CRUD 쿼리를 직접 작성할 필요가 없다.
  - 객체지향 프로그래밍을 쉽게 할 수 있음.
    - 부모 - 자식 관계 표현, 1:N 관계 표현, 상태와 행위를 한곳에서 관리하는 등.



### 요구사항 분석

-   게시판의 요구사항은 다음과 같다.
    - 게시판 기능
      - 게시글 조회
      - 게시글 등록
      - 게시글 수정
      - 게시글 삭제
    - 회원 기능
      - 구글 / 네이버 로그인
      - 로그인한 사용자 글 작성 권한
      - 본인 작성 글에 대한 권한 관리

## 3.2 프로젝트에 Spring Data Jpa 적용하기

### 1. build.grdle에 다음과 같이 의존성 등록한다.

- org.springframework.boot:spring-boot-starter-data-jpa

  - spring-boot-starter-data-jpa
    - 스프링 부트용 Spring Data Jpa 추상화 라이브러리.
    - 스프링 부트 버전에 맞춰 자동으로 JPA관련 라이브러들의 버전을 관리해 줌.

- com.h2database:h2

  - 인메모리 관계형 데이터베이스
  - 별도의 설치가 필요 없이 프로젝트 의존성만으로 관리할 수 있다.
  - 메모리에서 실행되기 때문에 애플리케이션을 재시작할 때마다 초기화된다는 점을 이용하여 테스트 용도로 많이 사용됨.
  - 이 책에서는 JPA의 테스트, 로컬 환경에서의 구동에서 사용할 예정.

  ```java
  /* build.gradle */
  dependencies {
      compile('org.springframework.boot:spring-boot-starter-web')
      compile('org.projectlombok:lombok') //롬복 추가
      compile('org.springframwork.boot:spring-boot-starter-data-jpa')// 스프링 부트용 Spring Data Jpa 추상화 라이브러리.
      complie('com.h2database:h2') // 인메모리 관계형 데이터베이스
      testCompile('org.springframework.boot:spring-boot-starter-test') 
  }
  
  
  ```



### 2. 의존성이 등록되었다면, 본격적으로 JPA기능을 사용해 보자.

#### 	2-1. 다음과 같이 domain 패키지를 만든다.

<img src="https://github.com/seulgi7/Book-Log/blob/2c4396d45d4c8a46fa82f87fdd99b1c94f5da18f/Spring/springboot_aws_webService/img/3.2_domain%E1%84%91%E1%85%A2%E1%84%8F%E1%85%B5%E1%84%8C%E1%85%B5.png" style="zoom:50%;" />

- domain 패키지 : 도메인을 담을 패키지

  >  도메인? 게시글, 댓글, 회원, 정산, 결제 등 소프트웨어에 대한 요구사항 혹은 문제영역이라 생각하면 됨.

#### 	2-2. domain 패키지에 posts 패키지와 Posts 클래스를 만든다.						

```java
package com.spring.book.springboot.domain.posts;

import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.*;

@Getter
@NoArgsConstructor
@Entity 
public class Posts {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 500, nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;

    private String author;

    @Builder
    public Posts(Long id, String title, String content) {
        this.id = id;
        this.title = title;
        this.content = content;
    }
}

```

>어노테이션 정렬 기준
>
>- 필자는 어노테이셔 순서를 **주요 어노테이션을 클래스에 가깝게**둔다.<br>
>  ex) @Entity는 JPA의 어노테이션이며 필수 어노테이션이지만, @Getter와 @NoArgsConstructor는 롬복의 어노테이션이며 필수 어노테이션은 아니다. 그러므로 주요 어노테이션인 @Entity를 클래스에 가깝게 두고, 롬복 어노테이션을 그 위로 두었다. 
>
> ➣ 이후에 코틀린 등의 새 언어 전환으로 롬복이 더이상 필요 없을 경우 쉽게 삭제할 수 있다.



##### **JPA에서 제공하는 어노테이션**

- **@Entity**

  - 테이블과 링크될 클래스임을 나타냄.
  - 기본값으로 클래스의 카멜케이스 이름을 언더스코어 네이밍(_)으로 테이블 이름을 매칭함.
  - ex) SalesManger.java → sales_manager table

- **@Id**

  - 해당 테이블의 PK 필드를 나타냄

    > - 웬만하면 Entry의 PK는 Long타입의 Auto_increment 추천.
    > - 주민등록번호, 복합키 등은 유니크 키로 별도로 추가하는 것을 추천.
    >   - 주민등록번호와 같이 비즈니스상 유니크 키나, 여러키를 조합한 복합키로 PK를 잡을 경우 난감한 상황이 종종 발생한다.
    >     1) PK를 맺을 때 다른 테이블에서 복합키 전부를 갖고 있거나, 중간 테이블을 하나 더 둬야 하는 상황 발생.
    >     2) 인덱스에 좋은 영향을 끼치지 못함.
    >     3) 유니크한 조건이 변경될 경우 PK 전체를 수정해야 하는 일이 발샘함.

- **@GenerateValue**

  - PK의 생성 규칙을 나타냄.
  - 스프링 부트 2.0에서는 GenerationTyoe.IDENTITY 옵션을 추가해야만 auto_increment가 됨.

- **@Column**

  - 테이블의 칼럼을 나타냄

  - 굳이 선언하지 않더라도 해당 클래스의 필드는 모두 칼럼이 됨.

  - 사용하는 이유는, 기본값 외에 추가로 변경이 필요한 옵션이 있으면 사용함.

    - ex) 문자열의 경우 VARCHAR(255)가 기본값인데, 사이즈를 500으로 늘리고 싶거나, 타입을 TEXT로 변경하거 싶은 등의 경우에 사용.

    

##### **롬복 라이브러리의 어노테이션**

서비스 초기 구축단계에선 테이블 선계가 빈번하게 변경되는데, 이때 롬복의 어노테이션들은 코드 변경량을 최소화시켜 주기 때문에 적극적으로 사용함.

- **@NoArgsConstructor**

  - 기본 생성자 자동 추가.
  - public Posts(){}와 같은 효과.

- **@Getter**

  - 클래스 내 모든 필드의 Getter 메소드를 자동생성.

- **@Builder**

  - 해당 클래스의 빌더 패턴 클래스를 생성.

  - 생성자 상단에 선언 시 생성자에 포함된 필드만 빌더에 포함.

    

##### 이 Posts 클래스의 한 가지 특이점

- Setter 메소드가 없다.

  - **Entire 클래스**에서는 **절대 Setter 메소드를 만들지 않는다.**
    - Why? 자바빈 규약을 생각하면서 getter/setterf를 무작정 생성하게 되면, 해당 클래스의 인스턴스 값들이 언제 어디서 변경해야하는지 코드상으로 명확하게 구분할 수가 없어, 차후 기능 변경 시 정말 복잡해짐.

- 대신, 해당 필드의 값 변경이 필요하면 명확히 그 목적과 의도를 나타낼 수 있는 메소드를 추가해야만 한다.

  - 잘못된 사용 예

    ```java
    public class Order{
      public void setStatus(boolean status){
        this.status = status
      }
    }
    
    public void 주문서비스의_취소이벤트(){
      order.setStatus(false);
    }
    ```

  - 올바른 사용 예

    ``` java
    public class Order{
      public void cancelOrder(){
        this.status = false;
      }
    }
    
    public void 주문서비스의_취소이벤트(){
      order.cancelOrder();
    }
    ```

    

##### 한 가지 의문 : Setter가 없는 이 상황에서 어떻게 값을 채워 DB에 삽입(insert)해야 할까?

- 기본적인 구조 : **생성자**를 통해 최종값을 채운 후 DB에 삽입하는 것.

  - 값 변경이 필요한 경우 해당 이벤트에 맞는 public 메소드를 호출하여 변경하는 것을 전제로 함.

- 이 책 :  생성자 대신 **@Builder**를 통해 제공되는 빌더 클래스를 사용.

  - 생성자와 @Builder의 비교

    - 공통점 : 생성 시점에 값을 채워주는 역할은 똑같음.

    - 차이점 : **생성자의 경우**만 지금 **채워야 할 필드가 무엇인지 명확히 지정할 수 없음.**

    - 예시 코드

      - **생성자** 이용

        - new Example(b,a)처럼  **a와 b의 위치를 변경해도 코드를 실행하기 전까진 문제를 찾을 수 없음.**

          ```java
          public class Example{
          	String a;
            String b;
            
            public Example (String a, String b){
              this.a = a;
              this.b = b;
            }
          }
          
          public void useExample(){
            new Example(a,b);
            //new Example(b,a); //이처럼 a와 b의 위치를 변경해도 코드를 실행하기 전까진 문제를 찾을 수 없음.
          }
          ```

      - **@Builder** 이용

        - **어느 필드에 어떤 값을 채워야 할지 명확하게 인지할 수 있음.**

          ```java
          Example.builder()
            .a(a)
            .b(b)
            .bulid();
          ```



#### 2-3. JpaRepository 생성

Posts 클래스 생성이 끝났다면, Posts 클래스로 Database를 접근하게 해줄 JpaRepository를 생성함.

<img src="https://github.com/seulgi7/Book-Log/blob/2c4396d45d4c8a46fa82f87fdd99b1c94f5da18f/Spring/springboot_aws_webService/img/3.2_jpaReposiitoy%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC.png" style="zoom:50%;" />



- **JPA**에선 **Repository**라고 부르며 **인터페이스**로 생성한다.

  - 단순히 인터페이스를 생성 후, JpaRepository<Entity 클래스, PK 타입>를 상속하면 기본적인 CRUD 메소드가 자동으로 생성됨.

    >  비교 : 보통 ibatis나 MyBatis 등에서 DAO라고 불리는 DB Layer 접근자다.

- **@Repository를 추가할 필요도 없다.**

  - 주의점 

    - Entity 클래스와 기본 Entity Repository는 **함께 위치**해야 함.
      - 둘은 아주 밀접한 관계이고, Entity 클래스는 기본 Repository 없이는 제대로 역할을 할 수가 없다.
    - Entity 클래스와 기본 Entity Repository는 함께 움직여야 하므로 **도메인 패키지에서 함께 관리**함.

    

## 3.3 Spring Data JPA 테스트 코드 작성하기.

#### 3-1. 테스트 클래스 생성

test 디렉토리에 domain.posts 패키지를 생성하고, 테스트 클래스는 PostsRepositoryTest란 이름으로 생성한다.

- 테스트 할 기능 : save, findAll

  - 예시 코드

    ```java
    package com.spring.book.springboot.web.domain.posts;
    
    import com.spring.book.springboot.domain.posts.Posts;
    import com.spring.book.springboot.domain.posts.PostsRepository;
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
    
        @After
        public void cleanup(){
            postsRepository.deleteAll();
        }
    
        @Test
        public void 게시글저장_불러오기(){
            //given
            String title = "테스트 게시글";
            String content = "테스트 본문";
    
            postsRepository.save(Posts.builder()
                    .title(title)
                    .content(content)
                    .author("seulgi")
                    .build());
            //when
            List<Posts> postsList =  postsRepository.findAll();
    
            //then
            Posts posts = postsList.get(0);
            assertThat(posts.getTitle()).isEqualTo(title);
            assertThat(posts.getContent()).isEqualTo(content);
        }
    }
    
    ```

  - @After

    - Junit에서 단위 테스트가 끝날 때마다 수행되는 메소드를 지정.
    - 보통은 배포 전 전체 테스트를 수행할 때 테스트간 데이터 침범을 막기 위해 사용함.
    - 여러 테스트가 동시에 수행되면 테스트용 데이터베이스인 H2dㅔ 데이터가 그대로 남아 있어 다음 테스트 실행 시 테스트가 실패할 수 있음.

  - @postsRepository.save

    - 테이블 posts에 insert/update 쿼리를 실행함.
    - id값이 있다면 update가, 없다면 insert쿼리가 실행됨.

  - @postsRepository.findAll

    - 테이블 posts에 있는 모든 데이터를 조회해오는 메소드.



## 3.4 등록/수정/조회 API 만들기

- API를 만들기 위해선 총 3개의 클래스가 필요하다

  1. Request 데이터를 받을 Dto

  1. API 요청을 받을 Controller

  1. 트랜잭션과 도메인 기능 간의 순서를 보장하는 Service
     - 도메인(Domain) : 비지니스 처리를 담당.

- update 기능의 주목할 점

  - update기능에서 데이터베이스에 쿼리를 날리는 부분이 없다.

  - 가능한 이유? JPA의 **영속성 컨텍스트**때문 

    > - 영속성 컨텍스트란 ? **엔티티를 영구 저장하는 환경.**
    > - JPA의 핵심 내용  : 엔티티가 **영속성 컨텍스트에 포함되어 있냐 아니냐**

    - 영속성 컨텍스트가 유지된 상태에서 해당 데이터의 값을 변경하면 **트랜잭션이 끝나는 시점에 **해당 테이블에 변경분을 반영함.

      즉, Entity 객체의 값만 변경하면 별도로 Updatd 쿼리를 날릴 필요가 없다는 것.   ➣  [더티 체킹](https://jojoldu.tistory.com/415)

      - 영속성 컨텍스트가 유지된 상태 ?  JPA의 엔티티 매니저*(EntityManager)*가 활성화된 상태로 *(Spring Data Jpa를 쓴다면 기본 옵션)*  **트랜잭션 안에서 데이터 베이스에서 데이터를 가져오면** 이 데이터는 영속성 컨텍스트가 유지된 상태.

        

## 3.5 JPA Auditing으로 생성시간/수정시간 자동화하기

- JPA Auditin 사용
  - 필요성
    - 보통 엔티티(Entity)에는 해당 데이터의 생성시간과 수정시간을 포함한다.
      - 언제 만들어졋는지, 언제 수정되었는지 등은 차후 유지보수에 있어 굉장히 중요한 정보이기 때문이다.
    - 매번 DB에 삽입*(insert)*하기 전, 갱신*(update)*하기 전에 날짜 데이터를 등록/수정하는 코드가 여기저기 들어가게 된다.
    - 이런 코드가 모든 테이블과 서비스 메소드에 포함되어야 한다고 생각하면 귀찮고 코드가 지저분해지는 결과를 낳는다.



### LocalDate사용 

- Java8부터 등장

  > Java8부터 LocalDate와 LocalDateTime이 등장함.
  >
  > - 그간 Java의 기본 날짜 타입인  Date의 문제점을 제대로 고친 타이이라 Java8이상일 경우 권장.
  >
  >   > ​	Date와 calendar클래스의 문제점
  >   >
  >   > 1. 불변객체가 아니다.
  >   >    - 멀티스레드 환경에서 언제든 문제가 발생할 수 있다.
  >   > 2. Calendar는 월(Month)값 설계가 잘못됨.
  >   >    - 10월을 나타내는 Calendar.OCTOBER의 숫자값을 '9'이다.
  >   >    - 당연히 '10'으로 생각했던 개발자들에게는 큰 혼란을  줌.



- 예제 코드

  1. **모든 Entity의 상위 클래스가 되는 클래스를 생성**해 **모든  Entity들의 createdDate, modified를 자동으로 관리하는 역할**을 하도록 한다.

     ```java
     package com.spring.book.springboot.domain;
     
     import lombok.Getter;
     import org.springframework.data.annotation.CreatedDate;
     import org.springframework.data.annotation.LastModifiedDate;
     import org.springframework.data.jpa.domain.support.AuditingEntityListener;
     
     import javax.persistence.EntityListeners;
     import javax.persistence.MappedSuperclass;
     import java.time.LocalDateTime;
     
     @Getter
     @MappedSuperclass
     @EntityListeners(AuditingEntityListener.class)
     public class BaseTimeEntity {
         @CreatedDate
         private LocalDateTime createdDate;
         
         @LastModifiedDate
         private LocalDateTime modifiedDate;
     }
     ```

     - @MappedSuperclass
       - JPA Entity 클래스들이 BaseTimeEntity를 상속할 경우 필드들(createdDate, modifiedDate)도 칼럼으로 인식하도로 함.
     - @EntityListeners
       - BaseTimeEntity 클래스에 Auditing 기능을 포함시킴.
     - @CreatedDate
       - Entity가 생성되어 저장될 때 시간이 자동 저장됨.
     - @LastModifiedDate
       - 조회환 Entity의 값을 변경할 때 시간이 자동 저장됨.

  2. Entity클래스가 해당 클래스를 **상속받도록 변경**한다.

     ```java
     	...
     public class Posts extends BaseTimeEntity {
     	...
     }
     ```

     

  3. Jpa Auditing 어노테이션을 모두 환성화할 수있도록 Application클래스에 **활성화 어노테이션을 추가**한다.

     ```java
     @EnableJpaAuditing // JPA Auditing 활성화
     @SpringBootApplication
     public class Application {
         public static void main(String[] args) {
             SpringApplication.run(Application.class);
         }
     }
     
     ```

     















