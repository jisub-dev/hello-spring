# Section 1. 프로젝트 환경 설정
* thymeleaf: HTML로 변경해주는 템플릿 엔진
* spring-boot-starter
  * 현업에서는 sout 으로 출력하면 안됨. 현업에서는 로깅을 활용해서 로그를 남겨야 한다 
  * slfj: 인터페이스 용도
  * logback: 원하는 타입으로 로깅할 수 있는 라이브러리
  * 테스트
    * junit 라이브러리를 많이 사용함
    * spring-test: 스프링과 통합해서 테스트 할 수 있게 해주는 라이브러리
* compileOnly("org.springframework.boot:spring-boot-devtools") 를 사용하면 html 파일을 컴파일만 해주면 서버 재시작 없이 View 파일 변경이 가능하다

# Section 2. 스프링 웹 개발 기초
* MVC: Model, View, Controller
* 요즘은 Controller과 View를 무조건 분리함 -> 유지보수 용이
* thymeleaf 의 장점은 파일 경로를 복사하여 실행하면 그대로 보임, 실제 서버를 통해 접근하면 ${} 안의 값이 바뀜
* 아래 코드에서 name의 값은 url 끝에 ?name=값 을 추가해야 name의 값을 전달할 수 있다
```java
@GetMapping("hello-mvc")
    public String helloMvc(@RequestParam("name") String name, Model model) {
        model.addAttribute("name", name);
        return "hello-template";
    }
```
* mvc, 템플릿 엔진 이미지 작동 원리
  1. 웹 브라우저에서 localhost:8080/hello-mvc?name=spring 서버에 요청
  2. 스프링 부트의 내장 톰캣 서버가 받아서 스프링 컨테이너의 helloController 호출 
  3. 컨트롤러가 hello-template에 model에 name:spring을 전달
  4. viewResolver 에서 templates/hello-template.html에 있는 정적파일에 Thymeleaf 엔진 처리를 함
  5. 변환된 html을 내장 톰캣 서버를 거쳐 웹 브라우저에 다시 반환
  6. 사용자는 hello spring! 이라는 화면을 보게 됨
* 아래와 같이 템플릿 엔진을 사용하지 않고 데이터를 반환하는 방식이 API임
```java
@GetMapping("hello-string")
    @ResponseBody
    public String helloString(@RequestParam("name") String name) {
        return "hello " + name; 
    }
```
* JSON 방식을 사용한 API
```java
// {"name":"spring!"} 을 반환함
@GetMapping("hello-api")
    @ResponseBody
    public Hello helloApi(@RequestParam("name") String name) {
        Hello hello = new Hello();
        hello.setName(name);
        return hello;
    }

    static class Hello {
        private String name;
        
        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
```
* Getter와 Setter를 사용하여 자원을 사용하는 방식을 Java Bean 방식이라고 한다
* Spring 에서 객체가 오면 기본적으로 JSON 방식을 사용해서 반환하는 정책이 적용되어 있음
* Spring은 단순 문자면 StringConverter(StringHttpMessageConverter)을 사용해 변환, 객체면 JsonConverter(MappingJackson2HttpMessageConverter)을 사용해 변환한다. 이 둘이 아니더라도 다른 converter가 기본적으로 등록되어 있다. 이를 웹 브라우저에 반환한다
* 