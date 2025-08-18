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
* 