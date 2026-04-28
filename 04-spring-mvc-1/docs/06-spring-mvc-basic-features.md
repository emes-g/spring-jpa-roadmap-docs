## 스프링 MVC 프로젝트 생성
### 템플릿 엔진 변경
- 기존 프로젝트(`servlet`)에서 사용하던 JSP 대신, 이번 프로젝트(`springmvc`)부터는 스프링 부트에서 권장되는 템플릿 엔진인 타임리프(Thymeleaf)를 사용한다.

### 패키징 방식 (WAR vs JAR)
#### WAR (Web Application Archive)
- **JSP를 사용하거나, 외부 서버를 사용할 때 주로 사용하는 전통적인 패키징 방식**이다.
  - 즉, 별도의 WAS를 설치해두고 그 곳에 빌드된 파일을 배포해야 할 때 주로 사용하는 방식이다.
  - 물론, 내장 톰캣에서도 동작하긴 하지만, 권장되지는 않는다.
- **정적 리소스와 뷰 템플릿을 관리하기 위해 `src/main/webapp` 경로를 사용**한다.

#### JAR (Java Archive)
- **내장 톰캣을 사용하여 자바 앱을 실행할 때 사용하는 패키징 방식**으로, 스프링 부트에서 권장하는 방식이다.
  - **`.jar` 파일 내부에는 컴파일된 클래스 파일, 의존성 라이브러리뿐만 아니라 내장 톰캣 구동에 필요한 파일들이 모두 하나로 압축**(패키징)되어 있다.
  - 그렇기 때문에 **배포 환경에서 별도의 웹 서버를 구축할 필요 없이, 단순히 `java -jar {jar 파일명}` 명령어만으로도 패키징된 자바 앱을 실행할 수 있는 것**이다. 
- **정적 리소스를 제공하기 위해 `src/main/resources/static` 경로를 사용**한다.

---

## 로깅
### 로깅 라이브러리 개요
- **운영 시스템에서는 `sout`을 통한 콘솔 출력 대신 로깅 라이브러리를 사용해야 한다.** (이유는 후술)
- 스프링 부트 라이브러리에는 여러 로깅 라이브러리가 기본적으로 포함되어 있다.
  
#### SLF4J
- 로깅 라이브러리를 통합해서 제공하는 **인터페이스**이다.

#### Logback
- SLF4J의 여러 구현체(Log4j, Log4j2, Logback 등) 중 **스프링 부트가 기본으로 채택하는 구현체**이다. (덕분에 가장 많이 사용된다.)

### @Controller와 @RestController의 차이
#### @Controller
- **반환값이 `String`이면 해당 문자열을 뷰의 논리적 이름으로 인식하여 `ViewResolver`를 찾고 뷰를 렌더링**한다.

#### @RestController
- **반환값이 `ViewResolver`를 거치지 않고, HTTP 응답 메시지 바디에 직접 입력**된다. 
- REST API 개발 시 주로 사용하는 컨트롤러이다.

### 로그 레벨과 설정
> **로그는 중요도에 따라 5가지 레벨로 나뉘며, 환경에 맞게 출력 레벨을 조절할 수 있다.**

#### 로그 레벨 (오름차순)
1. **TRACE:** 가장 상세한 디버깅 추적 정보
2. **DEBUG:** 개발 단계에서의 디버깅 정보
3. **INFO:** 운영 시스템에서 봐야 할 유의미한 정보 (스프링 부트 기본 설정)
4. **WARN:** 시스템 에러는 아니지만, 잠재적인 위험이나 경고 상태
5. **ERROR:** 치명적인 시스템 오류

#### 로그 레벨 설정 방법
- **`application.properties` 파일에서 패키지별로 로그 레벨을 세밀하게 설정**할 수 있다.
- 기본값은 `INFO`이므로 `TRACE`나 `DEBUG` 로그는 출력되지 않는다. 
- 따라서 필요한 경우에는 적절히 설정을 변경해야 한다.
- **권장 레벨:**
  - 로컬 PC (개인 작업 환경): `TRACE` or `DEBUG`
  - 개발 서버 (팀 테스트 환경): `DEBUG`
  - 운영 서버 (실제 서비스 환경): `INFO`

### 로그 출력 관례
#### 롬복 활용
- **클래스 레벨에 `@Slf4j` 어노테이션을 붙이면, 롬복이 컴파일 시점에 자동으로 로그 객체를 생성하고 초기화**해준다.
  - 즉, `Logger log = LoggerFactory.getLogger(getClass());` 코드를 자동으로 실행해 준다.

#### 파라미터 방식 사용
- **비권장: `log.debug("trace my log =" + name);`**
  - 자바 언어 특성상 메서드 호출 전에 `+` 연산이 먼저 발생한다.
  - **만약 현재 로그 레벨이 `INFO`라서 `DEBUG` 로그가 출력되지 않는 상황이라도, 문자열 연산은 무조건 실행되므로 여러 리소스(메모리, CPU 등)가 낭비**된다.
- **권장: `log.debug("trace my log = {}", name);`**
  - 파라미터(`{}`)를 넘기는 방식을 사용하면, 내부 로직에서 현재 로그 레벨을 먼저 평가한다.
  - **출력할 필요가 없는 레벨이라면 문자열 연산 자체를 수행하지 않으므로 성능 낭비를 사전에 방지**할 수 있다.

### 로깅 사용 이유
- 쓰레드 정보, 클래스 이름, 발생 시간 등 **유용한 부가 정보를 함께 확인**할 수 있고, 출력 포맷을 자유롭게 조정할 수 있다.
- 애플리케이션 코드를 전혀 수정하지 않고, **설정 파일만 변경하여 개발/운영 환경에 맞게 로그 출력 레벨을 조절**할 수 있다.
- 콘솔 출력뿐만 아니라 파일, 네트워크 등 **별도의 위치에 로그**를 남길 수 있다.
- 특히 파일로 남길 때는 일별, 용량별로 **로그 파일을 분할**(Rolling)할 수도 있다.
- 내부적으로 버퍼링, 멀티 쓰레드 환경 최적화 등이 되어 있기 때문에 **일반 출력(`sout`)보다 성능이 좋다.**

---

## 요청 매핑
### 요청 매핑 개요
- **클라이언트의 요청이 들어왔을 때, 해당 요청을 처리할 컨트롤러(핸들러)의 메서드와 연결해 주는 과정**이다.
- **`@RequestMapping`은 단순히 URL 경로뿐만 아니라** HTTP 메서드, 파라미터, 헤더, 미디어 타입 등 **다양한 조건을 조합하여 매핑**할 수 있다.
- 배열을 사용하여 여러 URL을 하나의 핸들러 메서드에 동시에 매핑하는 것도 가능하다.
  - 예) `@RequestMapping({"/hello-basic", "/hello-go"})`

### 경로 변수 (@PathVariable)
- **URL 경로의 일부를 템플릿화(변수화)하여, 해당 위치에 들어오는 값을 편리하게 조회**할 수 있는 기능이다.
- **최근 웹 API(RESTful API) 설계에서는 식별자를 쿼리 파라미터 대신 URL 경로에 직접 넣는 방식을 사용하는데, 이때 `@PathVariable`이 사용**된다.
  - 예) `/users/1` (1번 사용자 조회) → `@GetMapping("/users/{userId}")`
- 쿼리 파라미터와 마찬가지로 여러 개의 경로 변수를 한 번에 사용하는 것도 가능하다.
  - 예) `/users/{userId}/orders/{orderId}`

### 특정 조건 매핑 (사용도 낮음)
> **URL 경로와 HTTP 메서드 외에도, 특정 조건이 만족되어야만 매핑되도록 제약을 걸 수 있다.**

#### 파라미터 조건 매핑
- 특정 쿼리 파라미터가 존재해야만 호출된다.
- 예) `params="mode=debug"`

#### 헤더 조건 매핑
- 특정 요청 헤더가 존재해야만 호출된다.
- 예) `headers="mode=debug"`

### 미디어 타입 조건 매핑
> **HTTP 요청 헤더의 Content-Type과 Accept 값을 기준으로 매핑 조건을 제한한다.**

#### consumes
- **클라이언트가 보내는 페이로드(메시지 바디)의 데이터 타입을 제한할 때 사용**한다.
- **클라이언트의 HTTP 요청 헤더 중 `Content-Type`과 일치해야 매핑**된다.
- 예) `consumes = "application/json"`으로 설정하면, 클라이언트가 JSON 형식의 데이터를 보낼 때만 해당 핸들러가 호출된다.

#### produces
- **특정 미디어 타입을 선호하는 클라이언트에게 응답할 때 사용**한다.
- **클라이언트의 HTTP 요청 헤더 중 `Accept`와 일치해야 매핑**된다.
    - 즉, 클라이언트가 선호하는(응답받길 원하는) 미디어 타입을 지원하는 경우에만 응답이 가능하다.
- 예) `produces = "text/html"`로 설정하면, 클라이언트가 `text/html`을 선호한다고 보낸 경우에만 해당 핸들러가 매핑된다.
- **서버는 응답 메시지 헤더에 `Content-Type = {produces 속성 값}` 필드를 추가하여 전송**한다.
- **예시 상세:**
  ```java
  // @RestController
  @PostMapping(value = "/mapping-produce", produces = "text/html")
  public String mappingProduces() {
    log.info("mappingProduces");
    return "ok";
  }
  ```
  - 해당 핸들러가 호출됐다고 가정하자.
  - `produces = "text/html"`로 설정되어 있기 때문에, 응답 메시지 헤더에 `Content-Type = "text/html"` 필드가 추가될 것이다.
  - 그렇다고 해서, `@RestController`에 의해 작성된 페이로드 데이터(`ok`라는 문자열)가 정말 `<html>ok</html>`과 같이 HTML 태그로 변환되는 것은 아니다.
  - 단순히 페이로드 데이터를 클라이언트가 HTML 문서로 렌더링하면 된다는 의미이다.

---

## HTTP 요청 - 기본 및 헤더 조회
### 어노테이션 기반 컨트롤러의 유연성
- 스프링의 **어노테이션 기반 컨트롤러는 파라미터가 인터페이스로 정형화되어 있지 않다.**
- 따라서 **스프링이 지원하는 임의의 객체나 어노테이션을 파라미터로 선언하기만 하면 스프링이 알아서 값을 채워 넣어준다.**
- 즉, **필요한 것을 선언하면 스프링이 주입해 준다.**

### 주요 지원 파라미터
```java
@RequestMapping("/headers")
public String headers(HttpServletRequest request,
                      HttpServletResponse response,
                      HttpMethod method,
                      Locale locale,
                      @RequestHeader MultiValueMap<@NonNull String, String> headerMap,
                      @RequestHeader String host,
                      @CookieValue(value = "myCookie", required = false) String cookie
) {
    // 로그 출력
    return "ok";
}
```
- **HttpServletRequest / Response**: 서블릿 표준 요청/응답 객체
- **HttpMethod**: 현재 요청의 HTTP 메서드 정보
- **Locale**: 언어 설정을 포함한 로케일 정보 
- **@RequestHeader MultiValueMap**:
  - 모든 HTTP 헤더 조회
  - 하나의 키에 여러 값이 있을 수 있기 때문에 `MultiValueMap`을 사용
- **@RequestHeader**: 특정 HTTP 헤더 조회
- **@CookieValue**: 
  - 특정 쿠키 정보 조회
  - 필수 여부(`required`)나 기본값(`defaultValue`)과 같은 설정 지원

### @ModelAttribute와 BindingResult
#### @ModelAttribute (객체 바인딩)
- **클라이언트가 보낸 여러 요청 파라미터를 해당 객체의 필드로 자동으로 매핑(바인딩)해 주는 어노테이션**이다.
- 이름에서 알 수 있듯, 단순히 객체를 생성하고 값을 채우는 것을 넘어 **해당 객체를 자동으로 모델(Model)에 담아주기 때문에 뷰 템플릿 엔진에서 바로 꺼내 쓸 수 있다.**

#### @ModelAttribute 장점
```java
@Getter
@Setter
public class Member {
    private String name;
    private int age;
}

@RestController
public class MemberController {
    
    @PostMapping("/members")
    // public String addMember(@RequestParam String name, @RequestParam int age) {
    public String addMember(@ModelAttribute Member member) {
        // 출력
        return "ok";
    }
}
```
1. **OCP 준수:**
   - `@ModelAttribute`를 사용하지 않고, `@RequestParam`을 통해 일일이 사용자의 요청을 받아들인다고 생각해보자.
   - 만약 도메인 객체에 `grade`라는 새로운 필드 변수가 추가된다면, 컨트롤러의 핸들러 메서드 파라미터 역시 수정해야만 한다.
   - 하지만 `@ModelAttribute`를 사용하면 도메인 객체에 필드만 추가할 뿐, 컨트롤러를 수정할 필요가 없다.
2. **개발자 편의성:**
   - 매개변수의 순서에 의존하지 않고 개발할 수 있다.

#### BindingResult (검증 및 오류 정보)
- **`@ModelAttribute` 등으로 데이터를 바인딩할 때 발생한 오류 정보를 보관하는 객체**이다.
- 보통 숫자 타입 필드에 문자가 들어오는 등의 바인딩 에러나 입력값 검증(Validation) 실패 시 활용한다.
- **반드시 검증 대상 객체(예: `@ModelAttribute`가 붙은 파라미터) 바로 다음에 위치해야 한다.**
  - 참고로 대상 객체가 여러 개라면(파라미터로 여러 개의 객체를 받는다면), 각각의 대상 바로 뒤에 개별 선언해야 한다.
- **`BindingResult` 객체를 활용하면 컨트롤러에서 직접 오류를 처리**할 수 있게 된다.
  - 즉, 예외를 밖으로 던지지 않아도 된다.

---

## 클라이언트에서 서버로 데이터를 전달하는 방법
### 개요
#### 요청 파라미터 방식
- GET 방식의 쿼리 파라미터나 POST 방식의 HTML Form을 의미한다.
- 전달되는 데이터의 형식(Key-Value 파라미터 쌍)이 동일하기 때문에, **서버 입장에서는 동일한 방식(요청 파라미터 방식)으로 데이터를 조회**할 수 있다.
- **GET 방식의 쿼리 파라미터:**
  - URL의 쿼리 스트링을 통해 데이터를 전송하는 방식으로, HTTP 메시지 바디를 사용하지 않는다.
- **POST 방식의 HTML Form:**
  - HTML Form 태그를 사용하는 방식으로, 데이터를 HTTP 메시지 바디에 담아 전달한다.
  - 메시지 바디를 사용하기 때문에, `Content-Type` 값을 필수적으로 지정해야 한다. (`Content-Type: application/x-www-form-urlencoded`)
- 참고로, 파라미터의 키는 대소문자를 구분한다. (즉, `userName`과 `username`은 다르다.)

#### HTTP API
- **순수 데이터를 HTTP 메시지 바디에 담아 전달하는 방식**으로, 다양한 시스템 간의 통신에서 활용된다.
- 여러 방식(TEXT, XML, JSON, ...)이 있지만, 그 중에서도 주로 JSON이 활용된다.

### @RequestParam을 통한 요청 파라미터 조회
#### V1. 서블릿 기반 방식
```java
// @Controller
@RequestMapping("/request-param-v1")
public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {

    String username = request.getParameter("username");
    int age = Integer.parseInt(request.getParameter("age"));
    log.info("username={}, age={}", username, age);

    response.getWriter().write("ok");
}
```
- 과거 서블릿 시절과 동일하게 `HttpServletRequest`를 통해 파라미터를 조회하는 방식이다.
- 참고로 **`@Controller`더라도 반환값이 `void`면, 뷰 리졸버가 호출되지 않고 메시지 바디에 데이터가 전달**된다.

#### V2. @RequestParam 어노테이션 도입
- 스프링이 제공하는 `@RequestParam`을 사용하여 요청 파라미터를 메서드의 매개변수에 직접 바인딩한다.
  - 예) `@RequestParam("username") String memberName`
- 그리고 앞서, `@RestController`가 `@Controller`와 `@ResponseBody`의 조합이라고 했던 것을 기억해 보자.
  - 이에 기반하자면, **뷰의 논리적 이름을 반환하는 방식과 데이터를 응답 메시지에 바로 작성하는 방식을 혼용하고 싶은 경우**에는 어떻게 해야 할까?
  - 결론만 말하자면 **클래스 레벨은 `@Controller`로 해두고, 응답 메시지에 바로 작성하고 싶은 메서드 핸들러에만 `@ResponseBody`를 등록**하면 된다. 
    
#### V3. 요청 파라미터명 생략
- HTTP 요청 파라미터 이름과 컨트롤러 메서드의 변수 이름이 완전히 일치하면 `@RequestParam`의 괄호와 이름을 생략할 수 있다.
  - 예) `@RequestParam String username`

#### V4. 어노테이션 생략
- 요청 파라미터명뿐만 아니라 `@RequestParam`이라는 어노테이션 자체도 생략할 수 있다.
- 하지만 해당 파라미터가 HTTP 요청 데이터를 읽어온다는 사실을 명확히 하기 위해 V3 방식처럼 **`@RequestParam`을 명시하는 것이 유지보수 관점에서 더 좋다.**

#### 파라미터 필수 여부 설정 (required 속성)
- 기본적으로 `@RequestParam`은 `required` 속성 값이 `true`로 설정되어 있다.
- 따라서 **`@RequestParam`이 설정된 파라미터를 누락할 경우, 클라이언트 오류인 `400 Bad Request` 예외가 발생**한다.

#### 기본값 설정 (defaultValue 속성)
- `@RequestParam`에서는 값이 누락될 경우에 대비해, 기본값을 설정할 수 있다.
- **`null`과 Primitive 타입:**
  - 값이 안 들어오면 `null`이 할당되어야 하는데, Primitive 타입 변수에는 `null`을 입력할 수 없어 `500` 에러가 발생한다.
  - 따라서 Primitive 타입 변수를 Wrapper 클래스로 변경하거나, `defaultValue` 속성을 사용하여 기본값을 지정해야 한다.

#### Map을 이용한 전체 파라미터 조회
```java
@ResponseBody
@RequestMapping("/request-param-multi-map")
public String requestParamMultiMap(@RequestParam MultiValueMap<String, Object> paramMap) {

    log.info("username={}, age={}", paramMap.get("username"), paramMap.get("age"));
    return "ok";
    
    // username=[kim, choi], age=[18]
}
```
- 요청 파라미터를 단일 변수가 아닌 `Map` 형태로 한 번에 받을 수 있다.
  - 예) `@RequestParam Map<String, Object> paramMap`
- 동일한 키에 여러 값이 들어올 수 있는 상황(예: 다중 선택 체크박스)이라면 `MultiValueMap`을 사용해야 한다.
  - 하지만 실무에서 파라미터 값이 여러 개 들어오는 경우는 흔치 않다.

### @ModelAttribute를 통한 요청 파라미터 조회
#### @ModelAttribute (객체 바인딩)
- **클라이언트가 보낸 여러 요청 파라미터를 해당 객체의 필드로 자동으로 매핑(바인딩)해 주는 어노테이션**이다.
- 이름에서 알 수 있듯, 단순히 객체를 생성하고 값을 채우는 것을 넘어 **해당 객체를 자동으로 모델(Model)에 담아주기 때문에 뷰 템플릿 엔진에서 바로 꺼내 쓸 수 있다.**

#### @Data
- 클래스 레벨에 선언하는 어노테이션으로, 다음의 어노테이션들을 자동으로 적용해 준다.
  - `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode`, `@RequiredArgsConstructor`
- 무분별하게 Setter를 생성하여 객체의 캡슐화를 깨뜨릴 위험이 있기 때문에, **엔티티 등 중요한 핵심 도메인 객체에서는 사용을 지양하고 DTO(데이터 전송 객체) 용도로만 제한적으로 사용**해야 한다.

#### @ModelAttribute의 동작 원리
1. 스프링이 해당 객체(`HelloData`)를 자동으로 생성한다.
2. HTTP 요청 파라미터명과 동일한 객체의 프로퍼티(Getter/Setter)를 찾는다.
3. 해당 객체의 수정자 프로퍼티(Setter)를 호출하여 파라미터 값을 객체에 바인딩한다.
   - 가령 파라미터명이 `username`이면, `setUsername()` 메서드를 찾아 값을 주입한다.

#### 어노테이션 생략 규칙
- `@ModelAttribute` 역시 `@RequestParam`과 마찬가지로 컨트롤러 매개변수에서 생략할 수 있다.
- 스프링은 요청 파라미터 관련 어노테이션이 생략되었을 경우, 다음과 같은 규칙을 적용한다.
  - **원시 타입, 래퍼 클래스:** `@RequestParam`으로 인식
  - **사용자 정의 객체:** `@ModelAttribute`로 인식
  - **스프링 제공 특정 객체:** 요청 파라미터를 바인딩하는 대신, 해당 객체 자체를 파라미터에 주입 (내부적으로 `ArgumentResolver`가 동작하여 처리)

### HTTP API - 단순 텍스트 형식의 메시지 바디 조회
> **클라이언트가 요청 메시지 바디에 단순 텍스트 데이터를 담아 전달할 때, 해당 데이터를 서버가 어떻게 조회하는지 살펴보자.**

#### V1. 서블릿 기반 방식
```java
@PostMapping("/request-body-string-v1")
public void requestBodyString(HttpServletRequest request, HttpServletResponse response) throws IOException {
    ServletInputStream inputStream = request.getInputStream();  // 바이트 스트림으로 꺼내서
    String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8); // 인코딩
    
    response.getWriter().write("ok");
}
```
1. `request.getInputStream()`을 호출하여, 컨텐츠를 바이트 스트림으로 꺼낸다
2. 스프링이 제공하는 `StreamUtils.copyToString()`을 사용하여 바이트 스트림을 문자열로 변환한다.
    - 이때 인코딩 규격을 명시해야 한다.
    - 인코딩(바이트 ↔ 문자)할 때 인코딩 규격(기준이 되는 문자표, 주로 `UTF-8`)을 명시하지 않으면 데이터가 깨지기 때문이다.

#### V2. 스트림 객체 주입
- **서블릿 요청/응답 객체 대신, `InputStream`이나 `Writer`를 파라미터로 직접 주입받아 사용**한다.
- 앞서 어노테이션 생략 규칙에서 확인했듯, **스프링 제공 객체이므로 내부적으로 `ArgumentResolver`가 동작**한다.

#### V3. HttpEntity 도입
- **바이트 스트림 추출뿐만 아니라 인코딩 과정까지 스프링이 대신 처리**한다.
- **파라미터로 `HttpEntity<String>`을 받으면 내부적으로 `HttpMessageConverter`가 동작하여 메시지 바디를 문자로 변환**해 준다.
- **HttpEntity:**
  - HTTP 헤더와 메시지 바디를 편리하게 조회/조작할 수 있는 객체이다.
  - 요청 파라미터 조회(`@RequestParam`, `@ModelAttribute`) 기능과는 완전히 무관하다.
- **응답 시에도 `HttpEntity`를 반환할 수 있는데, `HttpEntity`를 반환하면 뷰 리졸버를 거치지 않고 응답 메시지 바디에 직접 작성**하게 된다.
- **ResponseEntity:**
  - `HttpEntity`를 상속받은 객체로, **메시지 바디뿐만 아니라 HTTP 상태 코드와 응답 헤더를 동적으로 제어**할 수 있다.
  - **성공 시 기본적으로 `200 OK`를 반환하는 `@ResponseBody`와 달리, `ResponseEntity`는 동적으로 상태 코드를 결정할 수 있으며, 커스텀 헤더를 추가하는 과정 또한 용이하기 때문에 자주 사용**된다.

#### V4. @RequestBody, @ResponseBody 도입
- **파라미터에 `@RequestBody`를 명시하면 내부적으로 `HttpMessageConverter`가 동작하여 메시지 바디를 직접 읽고 원하는 타입(텍스트의 경우 `String`)으로 변환**해 준다.
- **응답 시에는 `@ResponseBody`를 사용하여 뷰를 거치지 않고, HTTP 응답 메시지 바디에 결과를 직접 작성**할 수 있다.
  - 앞서 말했지만, `ResponseEntity` 또한 여러 장점(상태 코드 동적 제어, 커스텀 헤더 추가 용이)을 가지고 있기 때문에 `ResponseEntity`와 `@ResponseBody` 모두 알아둘 필요가 있다.

### HTTP API - JSON 형식의 메시지 바디 조회
> **클라이언트가 요청 메시지 바디에 JSON 데이터를 담아 전달할 때, 해당 데이터를 서버가 어떻게 조회하는지 살펴보자.**

#### V1. 서블릿 기반 방식 + ObjectMapper
```java
@PostMapping("/request-body-json-v1")
public void requestBodyJsonV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
    ServletInputStream inputStream = request.getInputStream();  // 바이트 스트림 꺼내서
    String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8); // 인코딩

    HelloData helloData = objectMapper.readValue(messageBody, HelloData.class); // 역직렬화 (JSON to 객체)

    response.getWriter().write("ok");
}
```
1. `request.getInputStream()`을 호출하여, 컨텐츠를 바이트 스트림으로 꺼낸다
2. 스프링이 제공하는 `StreamUtils.copyToString()`을 사용하여 바이트 스트림을 문자열로 변환한다.
3. Jackson 라이브러리의 `ObjectMapper` 클래스를 활용하여 문자열을 객체로 역직렬화(`readValue()`)한다.
    - 반대로 직렬화할 경우에는 `writeValueAsString()`을 사용하면 된다.

#### V2. @RequestBody + ObjectMapper
- `@RequestBody`를 통해 메시지 바디를 문자열로 받은 뒤, **개발자가 직접 `ObjectMapper`를 호출하여 객체로 변환**한다. 
  - 예) `@RequestBody String messageBody`
- `@RequestBody`를 사용하므로, `HttpMessageConverter`가 내부적으로 동작한다고 볼 수 있다.

#### V3. @RequestBody 단독
- `@RequestBody`를 통해 메시지 바디를 받을 때, **처음부터 원하는 객체 타입으로 받는 방식**이다.
  - 예) `@RequestBody HelloData helloData`
- **문자열을 받을 때는 단순히 바이트 스트림을 꺼내서 인코딩하는 과정까지만 거쳤지만, 객체를 받기 때문에 역직렬화 과정까지 거친다.**
  - 애초에 두 경우에서 동작하는 `HttpMessageConverter`의 구현체가 다르다.
- 참고로, **`@RequestBody`는 생략해서는 안 된다.**
  - 앞서 어노테이션 생략 규칙에서 확인했듯, 사용자 정의 객체에서 `@RequestBody`를 생략할 경우 `@ModelAttribute`가 동작하여 바인딩에 실패하기 때문이다.

#### V4. HttpEntity 활용
- 단순 텍스트에서 `HttpEntity<String>`으로 텍스트 데이터를 받았던 것처럼, JSON에서는 `HttpEntity<객체타입>`으로 객체 데이터를 받을 수 있다.
- **`@RequestBody`, `@ResponseBody`와 마찬가지로 내부적으로 `HttpMessageConverter`가 동작**한다.

#### V5. @ResponseBody를 통한 JSON 응답
- **반환 타입으로 단순 텍스트뿐만 아니라 객체도 지정할 수 있다.**
- **`@ResponseBody`를 등록하면 뷰 리졸버 대신 `HttpMessageConverter`가 동작하며, 해당 객체를 JSON 포맷으로 직렬화하여 클라이언트에게 응답**하게 된다.
- 이때 클라이언트의 Accept 헤더가 JSON을 수용할 수 있는 상태여야 한다.
  - 예) `Accept: */*` 등

#### 흐름 요약
1. 클라이언트가 보낸 요청 메시지 바디를 읽기 위해 `@RequestBody`를 사용한다.
2. `@RequestBody`를 사용했으므로, 내부적으로 `HttpMessageConverter`가 동작하여 내가 원하는 타입(`String`, 객체 등)으로 데이터를 확보할 수 있다.
3. 뷰 리졸버를 거치지 않고, 응답 메시지를 직접 작성하기 위해서는 `@ResponseBody`를 사용한다.
4. `@ResponseBody`를 사용하면, 내부적으로 `HttpMessageConverter`가 동작하여 직렬화(객체 → 문자열)된 데이터를 반환하게 된다.
5. 응답 메시지를 작성하는 것 외에도, 동적으로 상태를 제어하거나 커스텀 헤더를 추가해야 한다면 `@ResponseBody` 대신 `ResponseEntity`를 사용하면 된다.

---