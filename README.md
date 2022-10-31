해당 REPO는 [스프링 부트와 AWS로 혼자 구현하는 웹 서비스](http://www.yes24.com/Product/Goods/83849117) 교재를 바탕으로 공부하였습니다.

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

# 2 test-code

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
