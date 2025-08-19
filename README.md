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

# Section 3. 회원 관리 예제 - 백엔드 개발 
* 만약 store.get(id)가 null을 반환할 가능성이 있다면 Optional.ifNullable(store.get(id)) 처럼 감싸면 null값도 반환할 수 있게 된다
* 자바 실무에서는 루프를 돌리기 편하므로 리스트를 많이 사용함
* Assertions.assertThat(비교대상1).isEqualsTo(비교대상2) 이렇게 테스트를 수행한다
* 테스트 코드를 여러개 만들면 서로 테스트에 간섭할 수 있으므로 @AfterEach를 사용하여 매 테스트마다 저장소를 비워줘야 서로의 테스트에 영향을 끼치지 않는다
* 테스트 코드 없이 개발은 거의 불가능하다. 규모가 커질수록 문제가 정말 많아지기 때문이다
* Optional 을 사용하면 객체를 한번 더 감싸기 때문에 예외를 판별하기 용이하다
* Service는 비즈니스에 맞춰서 네이밍 함
* 테스트는 과감하게 한글로 적어도 됨 (영어권과 협업하는게 아니라면 자주 씀)
* given-when-then 기법을 추천함. 주로 이렇게 구성됨
* 테스트는 예외가 발생하는것을 테스트 케이스로 만드는 것이 중요함
* @BeforeEach: 매 테스트 전 실행. 강의에선 Service, Repository 생성 시 사용
* @AfterEach: 매 테스트 후 실행. 강의에선 메모리를 지워줄 때 사용
* DI (의존성 주입): 서비스 객체가 자신이 의존하는 리포지토리 객체의 구현체를 직접 생성하지 않고, 외부에서 전달받아 사용하는 설계 방식
* 리포지토리는 데이터 접근(저장, 조회)에 집중
* 서비스 계층은 리포지토리를 활용하려 비즈니스 로직을 수행

# Section 4. 스프링 빈과 의존관계
* 컨트롤러의 생성자에 @Autowired를 사용하면 스프링이 연관된 객체를 스프링 컨테이너에서 찾아서 넣어준다. 이를 DI(의존성 주입)이라고 한다
* @Component 애노테이션이 있으면 스프링 빈으로 자동 등록됨. 이때 패키지에 포함되지 않은 파일들은 등록되지 않는다
* 스프링에서는 스프링 컨테이너에 스프링 빈을 등록할 때 대부분 싱글톤으로 등록한다
* 애노테이션이 아닌 자바로 직접 스프링 빈을 등록할 수 있다
```java
public class SpringConfig {
    
    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }
    
    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
}
```
* 하지만 Controller는 어쩔 수 없이 @Autowired를 사용해야 한다
* 의존관계가 실행중에 동적으로 변하는 경우는 없으므로 생성자 주입을 사용해야 한다
* 직접 자바로 연결하려 Config 파일을 운용하는 경우 적은 코드 변경으로 관리할 수 있다

# Section 5. 회원 관리 예제 - 웹 MVC 개발
* 템플릿 엔진은 모델 데이터를 바인딩하여 값을 읽는다