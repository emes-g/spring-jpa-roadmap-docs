## 서블릿 프로젝트 초기 설정
### 프로젝트 초기 설정
- 서블릿 개발 자체는 스프링 부트 환경이 필수가 아니지만, **내장 WAS 제공 및 편리한 기본 환경 설정**에서의 이점 때문에 강의에서는 스프링 부트 기반으로 프로젝트를 생성한다.

#### `start.spring.io`에서 설정 시
- `SNAPSHOT`, `M2` 등은 정식 릴리즈 버전이 아니므로 제외하고 안정된 버전을 선택한다.
- `Group`과 `Artifact`는 스프링 부트 프로젝트 생성 시, **현재 프로젝트를 고유하게 식별하기 위해 지정하는 필수 정보**이다.
- **Group:**
  - **프로젝트를 생성하는 조직, 기업 또는 개인의 고유한 식별자**이다.
  - 일반적으로 도메인 네임을 역순으로 작성하는 관례(Reverse Domain Name)를 따른다.
  - 예) `com.naver`, `org.springframework`, `hello` (단순 학습용)
- **Artifact:**
  - **해당 그룹 내에서 만들어지는 구체적인 프로젝트(모듈)명**이다.
  - **빌드 완료 후 생성되는 최종 결과물(`.jar` 또는 `.war` 파일)의 이름**이 되며, 보통 소문자와 하이픈(`-`)으로만 작성한다.
  - 예) `member-service`, `servlet`
- **Package Name:**
  - `Group`과 `Artifact` 값을 설정하면, 자바의 기본 패키지 경로(`Pacakge name`)가 자동으로 구성된다.
  - 예) Group이 `hello`이고, Artifact가 `servlet`이면, 자바 소스 코드가 위치할 최상위 기본 패키지명은 `hello.servlet`이 된다.

### 패키징 방식
> **빌드된 애플리케이션의 최종 배포(압축) 형태를 결정하는 설정**이다.

#### jar (Java Archive)
- 스프링 부트에서 기본적으로 권장하는 최신 방식이다.
- **WAS를 띄우는(구동하는) 라이브러리 자체를 `.jar` 압축 파일 안에 포함하여 빌드**한다. 
- 따라서 배포 환경에서 별도의 WAS 서버를 설치할 필요 없이, **단순히 `java -jar application.jar` 명령어만 실행하면 자바 실행 환경(JRE) 위에서 자체적으로 서버가 구동**된다.

#### war (Web Application Archive)
- 전통적인 자바 웹 애플리케이션 배포 표준 방식이다.
- **서버 장비에 별도의 WAS(외부 톰캣 등)를 미리 설치해 두고, 특정 폴더에 빌드된 `.war` 파일을 밀어 넣어서 배포할 때 사용**한다.
- **JSP를 사용할 때는 `war` 패키징 방식을 선택**해야 한다.
  - 스프링 부트에서도 기본적으로 `.war` 파일의 실행을 지원한다.
  - 하지만, 스프링 부트 환경에서 **JSP 뷰 템플릿을 구동하려면 톰캣이 JSP 파일을 컴파일하고 인식하기 위한 특수한 파일 경로 구조(`WEB-INF` 등)가 필요한데, 기본 `jar` 패키징은 이 경로 구조를 제대로 지원하지 못한다.**
  - 따라서, **JSP를 렌더링하려면 반드시 표준 웹 디렉터리 구조를 갖는 `war` 패키징을 선택**해야 한다.

### 내장 WAS
- 전통적인 서블릿 개발 방식은 별도로 WAS를 설치하고, 서블릿 코드를 클래스 파일로 컴파일하여 올린 후, 서버를 수동으로 실행해야 하는 번거로움이 있었다.
- 스프링 부트의 `Spring Web` 의존성을 추가하면 기본적으로 내장 톰캣(`Uses Apache Tomcat as the default embedded container`)을 제공한다.
- 덕분에 복잡한 인프라 설정이나 서버 설치 과정 없이, **IDE에서 자바의 `main()` 메서드 실행만으로 내장 톰캣을 띄워 편리하게 서블릿 코드를 실행**할 수 있다.

---

## Hello 서블릿
### 서블릿 등록 및 실행
#### @ServletComponentScan
- **스프링 부트 환경에서 서블릿을 직접 등록해서 사용할 때 `@SpringBootApplication` 클래스에 붙이는 어노테이션**이다.
- 스프링의 `@ComponentScan`이 하위 패키지에서 빈을 찾아 스프링 컨테이너에 등록하듯, `@ServletComponentScan`은 **서블릿 클래스를 찾아 내장 WAS의 서블릿 컨테이너에 자동으로 등록**해 준다.

#### 서블릿 구현과 매핑 (@WebServlet)
- **서블릿 클래스는 반드시 `HttpServlet`을 상속받아야 한다.**
- 클래스 레벨에 `@WebServlet(name = "helloServlet", urlPatterns = "/hello")` 어노테이션을 사용하여 **서블릿의 이름과 매핑될 URL 경로를 지정**한다.
  - 스프링 MVC의 `@RequestMapping`과 유사하게 라우팅 역할을 하는 것을 알 수 있다.

#### service() 메서드 오버라이딩
- **클라이언트의 요청이 오면 서블릿 컨테이너가 해당(적절한) 서블릿의 `service()` 메서드를 호출**한다.
- 이때 IDE에서 오버라이딩(`ctrl` + `o`) 시, `public`이 아닌 `protected void service(HttpServletRequest, HttpServletResponse)`를 선택해야 한다.
  - `public` 메서드는 HTTP뿐만 아니라 다른 프로토콜까지 포괄하는 `ServletRequest`, `ServletResponse` 객체를 파라미터로 받는 반면,
  - `protected` 메서드는 **HTTP 전용 객체를 파라미터로 받기 때문에, HTTP 헤더나 세션 등 웹 특화 기능을 바로 사용**할 수 있어 훨씬 편리하기 때문이다.

### HTTP 요청 및 응답 메시지 처리
> **서블릿은 HTTP 메시지 파싱 등 네트워크/프로토콜 레벨의 복잡한 작업을 자동화하여, 개발자가 비즈니스 로직에만 집중할 수 있게 지원**한다.

#### 요청 파라미터 읽기
- `request.getParameter("username")`을 통해 쿼리 파라미터를 쉽게 읽어올 수 있다.

#### 응답 메시지 생성
```java
response.setContentType("text/plain"); // Content-Type 헤더 설정
response.setCharacterEncoding("utf-8"); // 문자 인코딩 설정
response.getWriter().write("hello " + username); // HTTP 응답 메시지 바디에 데이터 입력
```

#### 로깅 설정 (HTTP 메시지 확인)
- WAS가 받은 순수한 HTTP 요청 메시지(텍스트)를 콘솔에서 직접 확인하고 싶다면, `application.properties`에 다음 옵션을 추가하면 된다. (개발 환경에서만 사용)
  - `logging.level.org.apache.coyote.http11=trace`

### 서블릿 컨테이너 동작 흐름 정리
1. **스프링 부트를 실행하면 메모리에 내장 톰캣 서버(WAS) 객체가 생성**된다.
2. **톰캣 내부의 서블릿 컨테이너가 `@ServletComponentScan`을 통해 찾은 서블릿 인스턴스들을 미리 생성**해 둔다.
3. 웹 브라우저에서 HTTP 요청 메시지가 서버로 전달된다.
4. WAS는 해당 텍스트 메시지를 파싱하여 `HttpServletRequest`와 `HttpServletResponse` 객체를 새로 생성한다.
5. URL 매핑 정보를 바탕으로 적절한 서블릿의 `service()` 메서드를 호출하며 두 객체를 파라미터로 넘겨준다.
6. 서블릿에서 로직을 처리하고 응답에 필요한 정보를 `response` 객체에 세팅한다.
7. 로직이 끝나면 WAS는 `response` 객체의 정보를 바탕으로 최종 HTTP 응답 메시지를 생성하여 브라우저로 반환한다.

### 정적 파일 경로 (webapp)
- `src/main/webapp` 경로에 `index.html`, `basic.html` 등의 정적 파일을 생성하여 사용한다.
- 스프링 부트 내장 톰캣 환경(특히 `.war` 패키징)에서 `webapp` 경로는 자바 웹 애플리케이션의 기본 웹 루트(Web Root) 디렉터리로 동작한다.
- 스프링 MVC의 `/resources/static` 디렉터리가 정적 파일을 제공하듯이, **해당 경로에 위치한 파일들은 별도의 서블릿 매핑이 없어도 WAS가 클라이언트에게 정적 파일(HTML 등)로 그대로 내려주게 된다.**

### 현재 프로젝트를 최상위 루트로 열어두지 않은 경우, 정적 파일을 찾지 못하는 문제
#### @SpringBootApplication 클래스를 직접 실행하는 경우, /webapp 디렉터리의 정적 파일들을 찾지 못함
- 상위 폴더(예: `04-spring-mvc-1`)를 최상위 루트로 열어둔 상태에서, 내부에 있는 스프링 부트 메인 클래스(`ServletApplication.java`)를 직접 실행하면 `index.html`을 찾지 못하고 404 에러가 발생한다.
- 이는 **작업 디렉터리 불일치**가 원인이다.
  - IntelliJ의 내부 실행기(Runner)는 현재 에디터에 열려있는 최상위 폴더(`04-spring-mvc-1`)를 기준 작업 디렉터리(Working Directory)로 설정하여 내장 톰캣을 구동한다.
  - 내장 톰캣은 이 작업 디렉터리를 기준으로 웹 루트인 `src/main/webapp` 경로를 찾는다.
  - 결과적으로 톰캣은 `04-spring-mvc-1/src/main/webapp`을 탐색하지만, 실제 파일은 한 depth 아래인 `04-spring-mvc-1/servlet/src/main/webapp`에 존재하므로 파일을 찾지 못한다.

#### Gradle의 `bootrun` 태스크를 실행하여 해결함
- IntelliJ 우측 Gradle 탭에서 하위 프로젝트(`servlet`)의 `Tasks` → `application` → `bootRun`을 더블 클릭하여 실행하면 정상적으로 화면이 출력된다.
- **해결 원리 (정확한 기준 경로 인식):**
  - Gradle의 `bootRun` 태스크는 **IntelliJ가 열어둔 최상위 루트 경로와 상관없이, 해당 태스크가 속한 `build.gradle` 파일의 위치(`servlet` 폴더)를 작업 디렉터리로 고정하여 실행**한다.
  - 그 결과, 내장 톰캣이 정확히 `servlet/src/main/webapp` 경로를 웹 루트로 인식하게 되어 정적 파일(`index.html`)을 정상적으로 찾아 렌더링한다.

---

## HttpServletRequest에 담긴 정보
### HTTP 요청 메시지 파싱 정보
> **`HttpServletRequest` 객체는 HTTP 요청 메시지를 파싱하여 Start Line, Header, Body 및 기타 네트워크 부가 정보를 개발자가 편리하게 조회할 수 있도록 제공**한다.

#### Start Line 정보
```java
String method = request.getMethod();    // GET
String protocol = request.getProtocol();    // HTTP/1.1
String scheme = request.getScheme();    // http 
StringBuffer requestURL = request.getRequestURL();  // http://localhost:8080/request-header
String requestURI = request.getRequestURI();    // /request-header
String queryString = request.getQueryString();  // username=kim
boolean secure = request.isSecure();    // https 사용 유무
```
- **URL 문법 구조(`scheme://[userinfo@]host[:port][/path][?query][#fragment]`)를 기준으로 생각**해보자.
  - `request.getScheme()`: 리소스 접근 방법 (`http`, `https`) 
  - `request.getProtocol()`: 통신 규약 및 버전 (`HTTP/1.1`)
  - `request.getRequestURL()`: host, port, path를 포함한 전체 주소 (`localhost:8080/request-param`)
  - `request.getRequestURI()`: host, port를 제외한 리소스 식별 경로 (`/request-param`)
- **명명 규칙(Scheme, Protocol, URL, URI 등)이 직관적이진 않지만, 자바 서블릿 스펙에 정의된 표준이므로 명칭과 역할을 있는 그대로 받아들이고 사용**해야 한다.

#### Header 정보 (원형)
```java
request.getHeaderNames().asIterator()
                .forEachRemaining(headerName -> System.out.println(headerName + " = " + request.getHeader(headerName)));
```
- 위와 같이 **전체 헤더를 조회할 수도 있고, `request.getHeaderName()`을 통해 특정 헤더만 조회**할 수도 있다.
  - 예1) `request.getHeaderName(host) = localhost:8080`
  - 예2) `request.getHeaderName(accept-language) = ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7`

#### Header 정보 (가공)
- 서블릿은 **자주 사용하는 헤더를 쉽게 읽을 수 있도록 전용 메서드를 지원**한다.
- `request.getServerName()`, `request.getServerPort()`: `Host` 헤더 정보를 가공하여 반환한다.
  - 예) `request.getServerName() = localhost`
  - 예) `request.getServerPort() = 8080`
- `request.getLocale()`: `Accept-Language` 헤더를 파싱하여 가장 우선순위가 높은 언어 정보를 객체로 반환한다.
  - 예) `request.getLocale() = ko_KR`

#### Body 정보
- `request.getContentType()`: 클라이언트가 보낸 데이터의 형식(`Content-Type` 헤더)을 조회한다.
- `request.getContentLength()`: 바디 데이터의 전체 크기를 조회한다.
- `request.getParameter()`: GET 쿼리 파라미터나 POST Form 데이터를 조회한다.
  - 순수 텍스트나 HTTP API(JSON 객체)는 별도의 스트림을 통해 직접 읽어들여야 한다.

#### 기타 네트워크 정보 (부가 정보)
- **HTTP 표준 메시지 스펙에는 없지만, 실제 네트워크 소켓 연결을 통해 획득할 수 있는 부가 정보**이다. (거의 사용하지 않는다.)
- **Remote (요청을 보낸 클라이언트/프록시 정보):** 
  - `request.getRemoteHost()`
  - `request.getRemoteAddr()`
  - `request.getRemotePort()`
- **Local (요청을 받은 서버 정보):**
  - `request.getLocalName()`
  - `request.getLocalAddr()`
  - `request.getLocalPort()`

### 서블릿 제공 부가 기능
#### 임시 저장소 기능
- **`request.setAttribute(name, value)`와 `request.getAttribute(name)`을 통해 데이터를 저장하고 조회**한다.
- HTTP 요청이 서버에 들어와서 응답이 나갈 때까지만 생명주기가 유지(request scope)되므로, 주로 **MVC 패턴에서 컨트롤러와 뷰 사이의 데이터 전달 목적으로 사용**된다.

#### 세션 관리 기능
- **`request.getSession()`을 호출하여 사용자의 로그인 상태 등을 유지하는 세션 객체를 생성하거나 조회**할 수 있다.

### 시사점
- **`HttpServletRequest` 객체는 HTTP 요청 메시지 텍스트를 파싱하여 개발자가 다루기 편한 객체 형태로 감싸놓은 래퍼(Wrapper)에 불과**하다.
- 따라서 이 객체가 제공하는 다양한 기능들을 온전히 이해하고 활용하려면, **근간이 되는 HTTP 스펙과 메시지 구조 자체에 대한 이해가 선행**되어야 한다.

---