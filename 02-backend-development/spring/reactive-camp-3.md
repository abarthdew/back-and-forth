---
title: 1-3. Spring Reactive - Spring WebFlux
date: 2021-03-15
categories: [Back, Spring]
tags: [Back, Spring]
---

# 1-3. Spring Web Flux

> 💡 Spring 5.0<br>
> - 2016년 M1공개
> - M5 → RCI (2017년 6월)
> - 자바 8+

## 1) Spring-WebFlux

- 구 Spring-Web-Reactive(스프링 4) / (스프링의 웹을 담당하는 기본 모듈은 web MVC)
- 스프링 5의 메인 테마는 원래 JDK9이었는데 이제는 WebFlux로 바뀜

### 🔰 용도

- 비동기-논블로킹 리액티브 개발에 사용
- 효율적으로 동작하는 고성능 웹 애플리케이션 개발
- 서비스간 호출이 많은 마이크로서비스 아키텍처에 적합

> 💡 마이크로서비스 아키텍처<br>
> 하나의 웹 서버 애플리케이션이 다른 애플리케이션을 호출하고, 또 다른 애플리케이션을 호출하는 것처럼, 여러 서비스를 호출하는 등의 서비스 간 상호 협력을 통해 하나의 기능이 처리되도록 만드는 구조.

### 🔰 2가지 개발방식 지원

- 기존의 어노테이션을 이용한 @MVC 방식
    - `@Controller`, `@RestController`, `@RequestMapping`
- 새로운 함수형 모델(어노테이션에 의지하지 않고, 명시적 함수 이용)
    - `RouterFunction`, `HandlerFunction`

### 🔰 새로운 요청-응답 모델

- 서블릿 스택과 API에서 탈피
- 서블릿 API는 리액티브 함수형 스타일에 적합하지 않음
- 기본적으로 webFlux는 서블릿 기반이 아님(호환성은 가지고 있음)
- `HttpServletRequest`, `HttpServletResponse` (서블릿 기반) → `ServerRequest`, `ServerResponse` (추상화된 새로운 모델)

### 🔰 지원 웹 서버/컨테이너

- `서블릿을 지원하는 컨테이너` : Servlet 3.1+ (Tomcat, Jetty, ...) ; 서블릿 3.1+의 비동기-논블로킹 요청 처리 기능
- `서블릿과 상관없음` : Netty, Undertow ; 비동기-논블로킹 IO 웹 http 서버

## 2) 함수형 스타일 WebFlux `RouterFuncton` + `HandlerFunction`

### 🔰 스프링이 웹 요청을 처리하는 방식

✔ 먼저, 서버 프로그램이 웹을 처리하는 방식을 살펴보자.

- 요청 매핑
    - 웹 요청을 어느 핸들러에게 보낼지 결정
    - `URL`, `헤더` 정보 참고
    - `@RequestMapping` : 웹 요청을 어느 컨트롤러 메서드가 처리할 것인지 매핑하는 것을 결정.
- 요청 바인딩
    - 핸들러에 전달할 웹 요청 준비.
    - 날라온 http 요청 중, 핸들러 메서드의 파라미터로 무엇이 전달되게 할 것인가, 전달 시 어떤 변환이 일어나게 할 것인가.
    - `웹 URL` path, `헤더`-`쿠키` 정보, `바디`에 담겨져 오는 json, xml과 같은 자바 오브젝트 형태로 변환해서 전달하는 방식의 바인딩이 일어남.
- 핸들러 실행
    - 전달 받은 요청 정보를 이용해 로직을 수행하고 결과를 리턴
    - 바인딩을 통해 자바 오브젝트로 변환된 요청 정보를 전달받고, 내부 로직을 수행한 후 자바 오브젝트를 리턴.
- 핸들러 결과 처리(응답 생성)
    - 핸들러의 리턴 값으로 웹 응답 생성.
    - 리턴 값의 타입에 따라 다양한 종류의 리턴 값을 웹 응답으로 변환하는 작업이 스프링 MVC에서 일어남.

```java
@RestController
public class MyController { // (Rest)컨트롤러 클래스 등록
  @GetMapping("/hello/{name}") // 메서드 정의, 어노테이션을 사용해서 요청 매핑
  String hello(@PathVariable String name) { // @PathVariable : 요청 바인딩 / url path에서 {name} 부분을 찾아 변수에 바인딩 후 넘김
	return "Hello" + name; // 핸들러 로직 수행 - return
	// 스프링 MVC가 리턴값을 적절히 변환해서 리턴
	// 스프링 타입으로 리턴한다면? responseBody에 문자열이 그대로 들어감
  }
}
```

### 🔰 RouterFunction

- 함수형 스타일의 요청 매핑

```java
@FunctionalInterface // 람다식으로 표현 가능
public interface RouterFunction<T extends ServerResponse> {
  Mono<HandlerFunction<T>> route(ServerRequest request); // (ServerRequest request) : ServerRequest는 WebFlux 버전의 웹 요청을 대표하는 API
  // Mono<HandlerFunction<T>> : 웹 플럭스 버전의 웹 응답인 ServerResponse나 그 서브타입의 Mono 퍼블리셔를 리턴하는 HandlerFunction의 Mono 타입
  // 간단히, ServerResponse를 리턴하는 HandlerFunction(웹 응답을 리턴하는 함수)
}

// 웹 플럭스에서는 Handler역할을 HandlerFunction 타입으로 정의된 함수 안에서 담당
// RouterFunction에서 라우팅한다는 건 곧 매핑한다는 뜻. 서버 요청을 받아서 핸들러 펑션을 찾아주는 것.
// 그러므로, Mono<HandlerFunction<T>> : 웹 응답을 리턴하는 함수
```

### 🔰 HandlerFunction

- 함수형 스타일의 웹 핸들러(컨트롤러 메소드 하나가 핸들러 펑션 하나에 대응)
- 웹 요청을 받아 웹 응답을 돌려주는 함수

```java
@FunctionalInterface
public interface HandlerFunction<T extends ServerResponse> {
  Mono<T> handle(ServerRequest request);
}

// ServerRequest request 전달받음
// ServerResponse 타입을 Mono<T>로 감싸서 리턴
```

### 🔰 함수형 WebFlux가 웹 요청을 처리하는 방식

- `RouterFunction` : url 등 정보를 가지고 어떤 핸들러가 요청을 처리할 것인지 결정
    - 요청 매핑
- `HandlerFunction` : 요청을 바인딩, 핸들러 로직 수행, 결과를 처리해서 최종적으로 웹 응답을 만들어내는 것(상태코드를 200으로 줄지, 헤더 세팅은 뭘 해서 돌려줄지, 바디 세팅은 뭘 할지 등)
    - 요청 바인딩
    - 핸들러 실행
    - 핸들러 결과 처리(응답 생성)

> 🚩 WebFlux 함수형 Hello/{name} 작성<br>
> - 함수를 2개 만든다
>   - HandlerFunction을 먼저 만들고
>   - RouterFunction에서 path 기준 매핑을 해 준다

```java
@RestController
public class MyController {
  @GetMapping("/hello/{name}")
  String hello(@PathVariable String name) { // 핸들러펑션
	return "Hello" + name;
  }
}
```

> ✅ HandlerFunction 만들기

```java
// 위 코드의 핸들러펑션 부분이 이렇게 바뀜
HandlerFunction helloHandler = req -> {
  String name = req.pathVariable("name"); // request에서 pathVariable을 꺼내옴
  Mono<String> result = Mono.just("Hello" + name); // 핸들러 로직 적용 후 결과 값을 Mono에 담는다
  Mono<ServerResponse> res = ServerResponse.ok().body(result, String.class);
  // 웹 응답을 ServerResponse로 만든다
  // http 응답에는 응답코드(상태값), 헤더, 바디가 필요
  // ServerResponse.ok() : ServerResponse에는 응답코드를 리턴하는 빌더가 있으니 활용
  // .body(result, String.class) : 헤더와 바디를 세팅
  // Mono에 담긴 ServerResponse 타입으로 리턴
  return res;
}
```

```java
// 위 코드를 더 간결하게 작성
HandlerFunction helloHandler = req
  -> ok().body(fromObject("Hello" + req.pathVariable("name")));
// 함수형 방식에서는 req.pathVariable("name")처럼 pathVariable에서 name을 명시적으로 가져오는 방식을 씀.
// fromObject : 웹플럭스는 모든 리턴 결과를 Mono 또는 Flux 컨테이너에 담아서 리턴함. 그러므로, 오브젝트의 타입을 확인하고 Mono에 담아주는 static 메서드. 

// 로직의 결과 값을 바디(.body())에 담고 상태코드(.ok())를 추가해 웹 응답(ServerResponse)로 만든다.
// content-type이나 기타 헤더들은 스프링이 알아서 만들어줌
```

> ✅ RouterFunction 만들기

```java
RouterFunction router = req
  -> RequestPredicates.path("/hello/{name}").test(req) // 웹 요청 정보 중에서 url 경로 패턴 검사
  ? Mono.just(helloHandler) : Mono.empty();
  // Mono.just(helloHandler) : true일 경우, 즉 조건이 맞을 경우 핸들러 함수를 Mono에 담아서 반환
  // Mono.empty() : 조건에 맞지 않으면 빈 Mono 반환, 함수니까 뭐라도 반환해야 해야 하기 때문에 기재.
```

### 🔰 HandlerFunction과 RouterFunction 조합

> ✅ RouterFunctions.route(predicate, handler) : 이걸 사용하면 핸들러 펑션과 라우터 펑션을 결합할 필요 없이 좀 더 간결한 코드 작성 가능

```java
RouterFunction router = 
  RouterFunctions.route(
	ReuqestPredicates.path("/hello/{name}"), // RouterFunction의 매핑 조건을 체크하는 로직만 발췌
	  req -> ServerResponse.ok().body(fromObject("Hello" + req.pathVariable("name"))) // HandlerFunction
  );
```

### 🔰 RouterFunction의 등록

- RouterFunction 타입의 @Bean으로 만든다 : @Bean을 정의해 오브젝트를 bean으로 등록시켜서 사용.

```java
@Bean
RouterFunction helloPathVarRouter() {
  return route(
	RequestPredicates.path("/hello/{name}"),
	req -> ok().body(fromObject("Hello" + req.pathVariable("name")))
  );
}

// 어떤 configuration 클래스든 집어넣으면 스프링이 이 웹 요청을 처리하는 라우터 펑션과 핸들러 펑션을 등록해서 동작하도록 함.
```

> ❓ 원래 controller 빈 하나만 생성해서 그 안에 있는 많은 메서드를 골라 썼는데, 빈 하나에 요청을 처리하는 로직 하나만 넣을 수 있을까?

### 🔰 핸들러 내부 로직이 복잡하다면 분리한다

- 핸들러 코드만 람다 식을 따로 선언하거나
- 메소드를 정의하고 메소드 참조로 가져온다

```java
-- (1) 오브젝트 형식으로 정의
// 핸들러 펑션의 람다 바디를 블럭 형태로 만들어 분리시킨다
HandlerFunction handler = req -> { // 다른 bean 호출을 포함한 복잡한 로직을 담은 람다식
  String res = myService.hello(req.pathVariable("name"));
  return ok().body(fromObject(res));
};

return route(path("/hello/{name}"), handler); // 분리한 바디를 변수로 집어넣음
```

```java
-- (2) 클래스 형식으로 정의
// 더 좋은 방법 : 컨트롤러를 만들듯 핸들러를 별개의 클래스로 정의
@Componenet
public class HelloHandler {
  @Autowired MyService myService; // 서비스 받음
  
  Mono<ServerResponse> hello(ServerReqeust req) { // 앞의 람다식과 동일한 메소드 타입을 가진 메소드
	String res = myService.hello(req.pathVariable("name"));
	return ok().body(fromObject(res));
  }
  ...
}
```

```java
-- (3) 메서드 레퍼런스를 이용해서 정의
// 람다식은 메서드 타입(파라미터 타입, 리턴 타입, 예외)이 일치하면 일반 메서드와 호환해서 사용할 수 있음
@Bean
RouterFunction helloRouter(@Autowired HelloHandler helloHandler) {
  return route(path("/hello/{name}"), helloHandler::hello); // helloHandler::hello : 빈의 핸들러 메소드 레퍼런스
}
```

### 🔰 RouterFunction의 조합

- 핸들러 하나에 @Bean 하나씩 만들어야 하나?
- RouterFunction의 and(), andRoute()등으로 하나의 @Bean에 n개의 RouterFunction을 선언할 수 있다(체이닝 방식)

### 🔰 RouterFunction의 매핑 조건의 중복

- 타입 레벨 - 메서드 레벨의 @RequestMapping처럼 공통의 조건을 정의하는 것 가능
- RouterFunction.nest()

```java
public RouterFunction<?> routingFunction() {
  return nest(
	pathPrefix("/persion"), // 시작 경로(url이 /person으로 시작하는 조건을 공통으로
	// 이 뒤에 오는 라우터 펑션들은 기본적으로 앞에 /person이 붙음
	nest(accept(APPLICATION_JSON)), // accept는 조회용 : APPLICATION_JSON을 accept하는 공통조건 중첩
	// *위 두 조건은 중첩조건임
	route(GET("/{id}"), handler::getPerson) // 위 두가지를 조건으로 해서 /person/{id} 경로의 GET이면 getPerson 핸들러로 매핑
	.andRoute(method(HttpMethod.GET), handler::listPeople) // http 메서드가 GET이면 /person경로에 listPeople 핸들러로
  ).andRoute(POST("/").and(contentType(APPLICATION_JSON)), // 웹 요청이 POST면 /person 경로에 contentType이 APPLICATION_JSON이면 createPerson 핸들러로
  handler::createPerson);
}
```

## 3) WebFlux 함수형 스타일의 장점

- 모든 웹 요청 처리 작업을 명시적인 코드로 작성
    - 메서드 시그니처 관례(기존 MVC는 RequestMapping와 같은 어노테이션을 달고, 파라미터의 타입을 정하고, 바디를 JSON 컨버터가 컨버팅하는 등 관례를 모두 외어야 함)와 타입 체크가 불가능한 어노테이션에 의존하는 @MVC 스타일보다 명확
    - 정확한 타입 체크 가능, 관례 혼동으로 인한 잘못된 코드와 오류를 줄일 수 있음
- 함수 조합을 통한 편리한 구성, 추상화에 유리
- 테스트 작성의 편리함
    - 핸들러 로직은 물론이고 요청 매핑과 리턴 값 처리까지 단위테스트로 작성 가능(웹 테스트, Mock 등 통합테스트 필요 없음)

## 4) WebFlux 함수형 스타일의 단점

- 함수형 스타일의 코드 작성이 편하지 않으면 코드 작성과 이해 모두 어려움
- 익숙한 방식으로도 가능한데 뭐하러? ⇒ 기존의 @Controller + @RequestMapping을 그대로 사용할 수 있는 @MVC WebFlux 개발 방식

### 🔰 @MVC WebFlux

- 어노테이션과 메소드 형식의 관례를 이용하는 @MVC 방식과 유사
- 비동기 + 논블로킹 리액티브 스타일로 작성

### 🔰 ServerRequest, ServerResponse

- WebFlux의 기본 요청, 응답 인터페이스 사용
- 함수형 WebFlux의 HandlerFunction을 메서드로 만들었을 때와 유사
- 매핑만 어노테이션 방식을 이용

```java
// 기존의 전통적인 방식의 코드
@RestController
public static class MyController {
  @ReqeustMapping("/hello/{name}")
  Mono<ServerResponse> hello(ServerReqeust req) { // 요청 정보가 미리 바인딩되어 들어가는 것이 아닌, ServerReqeust로 들어감
  // Mono에 감싸진 ServerResponse로 응답하고 상태 코드와 바디 등을 명시적으로 선언해서 리턴
	return ok().body(fromObject(req.pathVariable("name")));
  }
}

// 메소드로 재정의된 HandlerFunction
Mono<ServerResponse> hello(ServerReqeust req) {
  return ok().body(fromObject(req.pathVariable("name")));
}

// 요청 매핑과 등록은 기존 @MVC 방식으로
@RestController
public static class MyController {
  @ReqeustMapping("/hello/{name}")
	...
  }
}
```

### 🔰 @MVC 요청 바인딩과 Mono/Flux 리턴 값

- 가장 대표적인 @MVC WebFlux 작성 방식
- 파라미터 바인딩은 @MVC 방식 그대로
- 핸들러 로직 코드의 결과를 Mono/Flux 타입으로 리턴

### 🔰 @MVC와 동일한 바인딩

- 경로 변수
- 커맨드 오브젝트
- 폼 오브젝트, 모델 애트리뷰트

```java
@GetMapping("/hello/{name}") // @MVC 스타일 매핑
Mono<String> hello(@PathVariable String name) { // @MVC에서 사용하던 요청 바인딩 그대로
  return Mono.just("hello" + name); // 리턴 값은 Mono/Flux로
}
```

```java
@RequestMapping("/hello")
Mono<String> hello(User user) { // User user : 커맨드 오브젝트, 모델 오브젝트 바인딩 / url 파라미터 또는 form-data
  return Mono.just("hello" + user.getName());
}
```

### 🔰 @RequestBody 바인딩 (JSON, XML)

- T
- Mono<T>
- Flux<T>

```java
@RequestMapping("/hello")
Mono<String> hello(@RequestBody Mono<User> user) { // @RequestBody User user : 웹 요청의 body를 MessageConverter에서 바인딩 / @MVC와 동일
  // Mono 컨테이너에 넣어서 가져옴
  return user.map(u -> "hello" + u.getName()); // Mono의 연산자를 사용해서 로직을 수행하고 Mono로 리턴
  // Mono로 가져오면 map등 편리한 연산 기능을 사용할 수 있음.
}
```

```java
// 오브젝트 하나로 결과를 받기 보다는, 스트림과 같은 형태를 클라이언트로부터 받을 수 있음
@PostMapping(value = "/hello")
Flux<String> hello(@RequestBody Flux<User> users) { // User의 스트림 형태로 요청을 전달
  return users.map(u -> "hello" + u.getName()); // User의 스트림 형태로 로직 수행
}
```

### 🔰 ResponseBody 리턴 값 타입

- T : 오브젝트를 바로 리턴
- Mono<T> : Mono에 담아서 리턴
- Flux<T> : Flux에 담아서 리턴
- Flux<ServerSideEvent> : http 스트리밍을 적용하고 싶을 때
- void : 리턴할 게 없을 때
- Mono<Void> : 작업을 완료했으나 아무것도 리턴하지 않겠다는 뜻

## 5) WebFlux와 리액티브 기술 `WebClient` + `Reactive Data`

### 🔰 WebFlux만으로 성능이 좋아질까?

- 비동기-논블로킹 구조의 장점은 블로킹 IO를 제거하는 데서 나옴
- HTTP 서버에서 논블로킹 IO는 오래 전부터 지원
- 뭘 개선해야 하나?

### 🔰 개선할 블로킹 IO

- 데이터 액세스 리포지토리
- HTTP API 호출
- 기타 네트워크를 이용하는 서비스

### 🔰 JPA - JDBC 기반 RDB 연결

- 현재는 답이 없음
- 블로킹 메서드로 점철된 JDBC API
- 일부 DB는 논블로킹 드라이버가 존재하지만 : @Async 비동기 호출과 CFuture를 리액티브로 연결하고 스레드풀 관리를 통해 웹 연결 자원을 효울적으로 사용하도록 만드는 정도
- JDK 10에서 Async JDBC가 등장할수도

### 🔰 Spring Data JPA의 비동기 쿼리 결과 방식

- 리포지토리 메소드의 리턴 값을 @Async 메소드 처럼 작성

```java
@Async
CompletableFuture<User> findOneByFirstname(String firstname);
// 스프링 데이터에서 async로 실행시켜줌
// 리포지토리 메서드를 비동기로 실행하고 결과를 CompletableFuture로 돌려 받는다
// @Async이므로 비동기 실행과 동시에 메서드 리턴
```

```java
// 위 코드를 WebFlux에서 받아서 Mono 형태로 리턴할 수 있음
@GetMapping
Mono<User> findUser(String name) {
  return Mono.fromCompletionStage(myRepository.findOnByFirstName(name));
  // myRepository.findOnByFirstName(name)는 CompletableFuture<User>를 리턴하는 리포지토리의 메서드
  // CompletableFuture의 비동기 결과는 Mono로 변환
  // 리포지토리에서 Stream<T>로 리턴한다면 Flux로도 변환 가능하나 블로킹됨
}
```

### 🔰 본격 리액티브 데이터 액세스 기술

- 스프링 데이터의 리액티브 리포지토리 이용
    - DB와 같은 리포지토리를 액세스하는 기구 중 리액티브(비동기-논블로킹) 방식으로 DB를 액세스 하고 결과를 가져오는 것 : `MongoDB`, `Cassandra`, `Redis`
    - 위 DB를 이용하면 DB를 액세스하는 코드를 완벽하게 논블로킹 방식으로 만들 수 있음
- ReactiveCrudRepository 확장
    - 모든 결과를 Flux, Mono 타입으로 리턴, 비동기-논블로킹 방식으로 결과를 돌려받을 수 있음.

```java
public interface ReactivePersonRepository extends ReactiveCrudRepository<Person, String> {
  // ReactiveCrudRepository<Person, String> : 리액티브 방식의 CRUD 메서드 지원
  Flux<Person> findByLastname(Mono<String> lasgname);
  // (1) 0-n개의 결과를 비동기-논블로킹 리액티브 방식으로 조회하도록 Flux<T>로 리턴
  @Query("{'firstname': ?0, 'lastname': ?1}") {
	Mono<Person> findByFirstnameAndLastname(String firstname, String lastname);
	// (2) 0-n개의 결과를 비동기-논블로킹 리액티브로 조회하도록 결과를 Mono<T>로 리턴해도 상관없음
  }
}

// 이 메서드를 호출하면 DB 액세스는 비동기-논블로킹 방식으로 백단에서 동작하고, 메서드는 바로 리턴됨.
// 핸들러 펑션은 바로 종료할 수 있게 됨.
```

```java
// 다른 결합 방법
@Autowired UserRepository userRepository;

@GetMapping
Flux<User> users() {
  return userRepository.findAll(); // Flux 형태의 user를 담아서 리턴
}
```

```java
@GetMapping
Flux<String> users() {
  return userRepository.findAll().map(u -> "hello" + u.getName());
  // 리포지토리 결과 Flux에 대해 추가 로직 적용
}
```

```java
public Mono<ServerResponse> getPerson(ServerReqeust request) {
  int personId = Integer.valueOf(request.pathVariable("id")); // path에서 id 값을 추출해서
  Mono<ServerResponse> notFound = ServerResponse.notFound().build();
  Mono<Person> personMono = this.repository.getPerson(personId); // 리포지토리에 가져옴
  return personMono.flatMap( // 응답을 만듬 : flatMap적용(스트림)
	person
	  -> serverResponse.ok().contentType(APPLICATION_JSON).body(fromObject  (person)) // 모노 타입으로 리턴
  ).switchIfEmpty(notFound); // 값을 가져오지 못했을 경우
  // flatMap을 처리할 때 personMono에 값이 없다면, 뒤의 람다식은 실행되지 않고 바로 switchIfEmpty로 넘어감
 }
```

### 🔰 논블로킹 API 호출은 WebClient

- AsyncRestTemplate의 리액티브 버전 : ListenableFuture 보다 효율적으로 사용할 수 있는 WebClient를 제공
- 요청을 Mono/Flux로 전달할 수 있고
- 응답을 Mono/Flux 형태로 가져옴

```java
@GetMapping("/webclient")
Mono<String> webclient() {
  return WebClient.create("http://localhost:8080")
	.get()
	.url("/hello/{name}", "Spring") // 주소에 name이란 파라미터를 넣어서
	.accept(MediaType.TEXT_PLAIN)
	.exchange()
	.flatMap(r -> r.bodyToMono(String.class)) // 결과를 가져와서
	.map(d -> d.toUpperCase()) // 대문자로 바꾼 다음
	.flatMap(d -> helloRepository.save(d)); // d에 저장하고 리턴
}
```

### 🔰 함수형 스타일의 코드가 읽기 어렵다면

- 각 단계의 타입이 보이지 않기 때문이다
- 타입이 보이도록 코드를 재구성하고 익숙해지도록 연습이 필요

```java
// 타입을 노출하는 코드로 분해
@GetMapping("/webclient")
Mono<String> webclient() {
  WebClient wc = WebClient.create("http://localhost:8080"); // 기준 url을 넣어 WebClient 생성
  UriSpec<RequestHeadersSpec<?>> uriSpec = wc.get();
  // UriSpec<RequestHeadersSpec<?>> : 다음 단계로 uri 설정 준비 / .get() : http 메서드 결정
  RequestHeadersSpec<?> headerSpec = uriSpec.uri("/hello/{name}", "Spring");
  // RequestHeadersSpec<?> headerSpec : uri 패턴과 파라미터로 uri 설정 / uriSpec.uri() : uri 패턴과 파라미터로 uri 설정
  RequestHeadersSpec<?> headerSpec2 = headerSpec.accept(MediaType.TEXT_PLAIN);
  // headerSpec.accept : 헤더 설정
  Mono<ClientResponse> res = headerSpec2.exchange();
  // Mono<ClientResponse> : ServerResponse와 유사한 구조의 응답 정보 / headerSpec2.exchange() : 요청을 응답으로 교환
  Mono<String> data = res.flatMap(r -> r.bodyToMono(String.class));
  // res.flatMap : Mono 데이터에 적용한 함수의 결과가 Mono 타입이기 때문에 flatMap을 적용해야 함. 아니면 Mono<Mono<String>>이 됨.
  // r.bodyToMono : 요청 바디를 String 타입으로 변환해서 Mono에 담아 리턴하는 함수
  Mono<String> upperData = data.map(d -> d.toUpperCase());
  // data.map : 데이터에 함수를 적용해서 변환
  return upperData.flatMap(d -> helloRepository.save(d));
  // 데이터를 리포지토리에 저장하고 결과를 리턴 / save : Mono 타입을 세이브 한 다음 그 타입을 그대로 받아서 flatMap한 후 리턴
}
```

### 🔰 비동기-논블로킹 리액티브 웹 애플리케이션의 효과를 얻으려면

- WebFlux
    
    + `리액티브 리포지토리`
    
    + `리액티브 원격 API 호출`
    
    + `리액티브 지원 외부 서비스` (메세지 큐 등등)
    
    + `@Async 블로킹 IO`
    
- 코드에서 블로킹 작업이 발생하지 않도록 Flux 스트림 또는 Mono에 데이터를 넣어서 전달 : 중간에 블로킹이 일어나는 코드를 쓰면 안 됨

# 색인과 출처
### 1-3. Spring WebFlux
- [영상 주소](https://www.youtube.com/watch?v=HKlUvCv9hvA&list=PLdHtZnJh1KdZ6NDO9zc9hF4tONDLTSEUV&index=3)
