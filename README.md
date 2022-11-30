해당 REPO는 [스프링 부트와 AWS로 혼자 구현하는 웹 서비스](http://www.yes24.com/Product/Goods/83849117) 교재를 바탕으로 공부하였습니다.

## 환경
- Java 8
- Gradle 4.x
- Spring Boot 2.1.x

# TDD 
테스트가 주도하는 개발
- 항상 실패하는 테스트를 먼저 작성 (Red)
- 테스트가 통과하는 프로덕션 코드 작성 (Green)
- 테스트가 통과하면 프로덕션 코드를 리팩토링 (Refactor)

## 단위테스트 이점
- 단위테스트틑 개발 단계 초기에 문제발견
- 리팩토링, 라이브러리 업그레이드 시 기존 기능이 올바르게 작동하는지 확인
- 기능에 대한 불확실성 감소
- 시스템에 대한 실제 문서 제공

## 왜 테스트 코드를 작성해야 하는가?
단위테스트를 배우기 전 개발방식
1. 코드작성
2. 프로그램 실행
3. API테스트 도구로 HTTP요청
4. 요청결과를 sout으로 확인
5. 결과가 다르면 tomcat을 중지하고 코드 수정

매번 2-5 과정이 반복되고 의미없는 시간 소요됨<br>
또한 사람의 눈으로 검증하지 않고 자동검증을 할 필요가 있음<br>
기존 기능이 잘 작동되는 것을 보장하게 해준다.
-> 자바의 테스트 프레임워크 => JUnit (해당 교재에서는 JUnit4사용)

# 2. Gradle -> Spring-boot

## Application

```java
package com.asher.book.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
프로젝트의 메인 클래스
- @SpringBootApplication으로 인해 스플이 부트 자동 설정, 스프링 Bean 읽기와 생성을 모두 자동으로 설정
- @SpringBootApplication이 있는 위치부터 설정을 읽어가기 때문에 이 클래스는 항상 프로젝트 최상단에 위치해야함.
- SpringApplication.run으로 인해 내장 WAS를 실행하며, 항상 서버에 톰켓을 설치할 필요가없음
- 내장 WAS사용을 권장하는데, **언제 어디서나 같은 환경에서 스프링 부트를 배포** 할수 있기 때문

## controller
```java
package com.asher.book.springboot.web;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController // 1
public class HelloController {

    @GetMapping("/hello") // 2
    public String hello(){
        return "hello";
    }
}
```

### (1) RestController
- 컨트롤러를 JSON으로 반환하는 컨트롤러로 만듦
- @ResponseBody를 각 메서드 마다 선언했던 것을 한번에 선언하는 것과 같음

### (2) GetMapping
- HTTP Method인 Get의 요청을 받을 수 있는 API를 만들어 줌

## HelloControllerTest

```java
package com.asher.book.springboot.web;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import static org.junit.Assert.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class) // 1
@WebMvcTest(controllers = HelloController.class) // 2
public class HelloControllerTest { 

    @Autowired // 3
    private MockMvc mvc; // 4

    @Test
    public void hello가_리턴된다() throws Exception{
        String hello = "hello";

        mvc.perform(get("/hello")) // 5
                .andExpect(status().isOk()) // 6
                .andExpect(content().string(hello)); //7
    }
}
```

### (1) RunWith(SpringRunner.class)
- 테스트를 진행할 때 JUnit에 내장된 실행자 외에 다른 실행자를 실행시킵니다.
- 여기서는 SpringRunner라는 스프링 실행자를 사용
- 즉, 스플이 부트 테스트와 JUnit사이에 연결자 역할

### (2) WebMvcTest
- 여러 스프링 테스트 어노테이션 중, WEB(spring mvc)에 집중할 수 있는 어노테이션
- 선언할 경우 @Controller, @ControllerAdvice등을 사용할 수 있음
- 단, @Service, @Component, @Repository등은 사용할 수 없음
- 여기서는 컨트롤러만 사용하기 때문에 선언함

### (3) Autowired
- 스프링이 관리하는 빈을 주입받음

### (4) private MockMvc mvc
- 웹 API를 테스트할 때 사용
- 스프링 MVC테스트의 시작점
- 이 클래스를 통해 HTTP, GET, POST등에 대한 API 테스트를 할 수 있다.

### (5) MVC.perform(get("/hello"))
- MockMvc를 통해 /hello주소로 HTTP GET 요청을 한다.
- 체이닝이 지원되어 아래와 같이 여러 검증 기능을 선언할 수 있다.

### (6) .andExpect(status().isOk())
- mvc.perform의 결과를 검증
- HTTP Header의 Status를 검증
- 흔히 알고 있는 200, 404, 500 등의 상태 검증
- 여기서는 200인지 아닌지 검증

### (7) .andExpect(content().string(hello))
- mvc.perform의 결과를 검증한다.
- 응답 본문의 내용을 검증한다.
- Controller에서 "hello"를 리턴하기 때문에 이 값이 맞는지 검증한다.

## 롬복
Project setting -> Build, Execution -> compiler -> annotation Processor -> Check! Enable annotation proecess

### HelloResponseDto

```java
package com.asher.book.springboot.web.dto;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter // 1
@RequiredArgsConstructor // 2
public class HelloResponseDto {
    private final String name;
    private final int amount;
}
```

### (1) @Getter
- 선언된 모든 필드의 get메서드를 생성

### (2) @RequiredArgsConstructor
- 선언된 모든 final 필드가 포함된 생성자를 생성해 줍니다.
- final이 없는 필드는 생성자에 포함되지 않습니다.

### HelloResponseDto test-code
```java
package com.asher.book.springboot.web.dto;

import org.junit.Test;

import static org.assertj.core.api.Assertions.assertThat;

public class HelloResponseDtoTest {

    @Test
    public void 롬복_기능_테스트(){
        //given
        String name = "test";
        int amount = 1000;

        //when
        HelloResponseDto dto = new HelloResponseDto(name, amount);

        //then
        assertThat(dto.getName()).isEqualTo(name); // 1, 2
        assertThat(dto.getAmount()).isEqualTo(amount);
    }
}
```
### (1) assertThat
- assertj라는 테스트 검증 라이브러리의 검증 메서드 입니다.
- 검증하고 싶은 대상을 메서드 인자로 받습니다.
- 메서드 체이닝이 지원되어 isEqualTo와 같이 메서드를 이어서 사용할 수 있습니다. 

### (2) isEqualTo
- assertj의 동등 비교 메서드 입니다.
- assertThat에 있는 값과 isEqualTo의 값을 비교해서 같을 때만 성공입니다. 

```java
@GetMapping("/hello/dto")
public HelloResponseDto helloDto(@RequestParam("name") String name, @RequestParam("amount") int amount) { // 1
    return new HelloResponseDto(name, amount);
}
```
### (1) RequestParam
- 외부에서 API로 넘긴 파라미터를 가져오는 어노테이션
- 여기서는 외부에서 name(@RequestParam("name"))이란 이름으로 넘긴 파라미터를 메소드 파라미터 name(String name)에 저장하게 된다.

```java
@Test
public void helloDto가_리턴된다() throws Exception {
    String name = "hello";
    int amount = 1000;

    mvc.perform(
            get("/hello/dto")
                    .param("name", name) // 1
                    .param("amount", String.valueOf(amount)))
                    .andExpect(status().isOk())
                    .andExpect(jsonPath("$.name", is(name))) //2
                    .andExpect(jsonPath("$.amount", is(amount)));
}
```
### (1) param
- API테스트할 때 사용될 요청 파라미터를 설정
- 단, 값은 String만 허용
- 그래서 숫자/날짜 등의 데이터도 등록할 때는 문자열로 변경해야만 가능

### (2) jsonPath
- JSON응답값을 필드별로 검증할 수 잇는 메서드
- $를 기준으로 필드명을 명시
- 여기서는 name과 amount를 검증하니 $.name, $.amount로 검증

# 3. Spring-boot + JPA

## JPA?
-> 지향하는 바가 다른 2개영역(객체지향 프로그래밍 언어와 관계형 데이터베이스)을 
중간에서 패러다임 일치를 시켜주기 위한 기술

```
// 1
implementation('org.springframework.boot:spring-boot-starter-data-jpa')
// 2 
implementation('com.h2database:h2')
```

### (1) spring-boot-starter-data-jpa
- 스프링 부트용 Spring Data jpa 추상화 라이브러리
- 스프링 부트 버전에 맞춰 자동으로 JPA관련 라이브러리들의 버전을 관리

### (2) h2
- 인메모리 관계형 데이터베이스
- 별도의 설치가 필요 없이 프로젝트 의존성만으로 관리할 수 있음
- 메모리에서 실행되기 때문에 애플리케이션을 재시작 할 때마다 초기화 된다는 점을 이용하여 테스트용도로 많이 사용

```java
package com.asher.book.springboot.domain.posts;

import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.*;

@Getter // 6
@NoArgsConstructor // 5
@Entity // 1
public class Posts {
    @Id // 2
    @GeneratedValue(strategy = GenerationType.IDENTITY) // 3
    private Long id;

    @Column(length = 500, nullable = false) // 4
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;

    private String author;

    @Builder // 7
    public Posts(String title, String content, String author){
        this.title = title;
        this.content = content;
        this.author = author;
    }
}
```

- 저자는 어노테이션 순서를 주요 어노테이션 클래스에 가깝게 둔다
- @Entity는 JPA의 어노테이션 이며, @Getter와 @NoArgsConstructor는 롬복의 어노테이션임
- 롬복은 코드를 단순화 시켜주지만 필수 어노테이션이 아니므로 주오 어노테이션인 @Entity를 클래스에 가깝게 두고 롬복 어노테이션을 그 위에 선언함
- 이렇게 하면 새 언어로 전환할 때 필요없는 어노테이션인 경우 쉽게 삭제가 가능하다. 
- **여기서 Posts클래스는 실제 DB의 테이블과 매칭될 클래스이며 보통 Entity클래스 라고 한다.**
- JPA를 사용하면 DB데이터에 작업할 경우 실제 쿼리를 날리는 것 보다, Entity클래스의 수정을 통해 작업함.


### (1) @Entity
- 테이블과 링크될 클래스를 나타냄
- 기본값으로 클래스의 카멜케이스 이름을 언더스코어 네이밍으로 테이블 이름을 매칭
- Ex) SalesManager.java -> sales_manager table

### (2) @Id
- 해당 테이블의 PK필드를 나타냅니다.

### (3) @GeneratedValue
- PK의 생성 규칙을 나타냄
- 스프링 부트 2.0에서는 GenerationType.IDENTITY옵션을 추가해야만 auto_increment가 된다

### (4) @Column
- 테이블의 칼럼을 나타내며 선언하지 않더라도 해당 클래스의 필드는 모두 컬럼이 된다.
- 사용하는 이유는, 기본값 외에 추가로 변경이 필요한 옵션이 있으면 사용함
- 문자열 경우 VARCHAR(255)가 기본값, 사이즈를 500으로 늘리고 싶거나(ex.title), 타입을 TEXT로 변경하고 싶거나 (ex:content)등의 경우에 사용함.

### (5) @NoArgsConstructor
- 기본 생성자 자동 추가
- public Posts() {}와 같은 효과

### (6) @Getter
- 클래스 내 모든 필드의 Getter메서드를 자동생성

### (7) @Builder
- 해당 클래스의 빌더 패턴 클래스를 생성
- 생성자 상단에 선언시 생성자에 포함된 필드만 빌더에 포함

### Tip
- Entity의 PK는 Long타입의 Auto_increment를 추천함 (Mysql기준 bigint 타입이 됨)
- 주민등록번호와 같이 비즈니스상 유니크 키나, 여러 키를 조합한 복합키로 PK를 잡을 경우 난감한 상황이 발생됨
1. FK를 맺을 때 다른 테이블에서 복합키 전부를 갖고 있거나, 중간 테이블을 하나 더 둬야 하는 상황 발생
2. 인덱스에 좋은 영향을 끼치지 못함
3. 유니크한 조건이 변경될 경우 PK 전체를 수정해야 하는 일이 발생함.
4. 주민번호, 복합키 등은 유니크 키로 별도로 추가하는 것이 좋음.

<br/>

- Posts클래스에 특이점이 있는데 setter메서드가 없다.
- 자바빈 규약을 생각하면서 getter/setter를 무작정 생성하는 경우가 있는데 이렇게 되면 해당 클래스의 인스턴스 값들이 언제 어디서 변해야 하는지 코드상으로 명확히 구분할 수 없어, 차후 변경시 복잡해진다.
- 그래서 Entity클래스에서는 절대 setter메서드를 만들지 않는다. 
- 대신, 해당 필드의 값 변경이 필요하면 명확히 그 목적과 의도를 나타낼 수 있는 메서드를 추가해야한다. 

- 예를들어 주문 취소 메서드를 만든다고 가정하고 예시를 살펴보자.

잘못된 사용 예시
```java
public class Order{
    public void setStatus(boolean status){
        this.status = status;
    }
}

public void 주문서비스의_취소이벤트 () {
    order.setStatus(false);
}
```
올바른 사용 예시
```java
public class Order{
    public void cancelOrder(){
        this.status = false;
    }
}

public void 주문서비스의_취소이벤트 () {
    order.cancelOrder();
}
```

