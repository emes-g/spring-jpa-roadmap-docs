## 스프링에서 웹을 개발하는 3가지 주요 방식

### 1. 웹 브라우저 요청 시 스프링 부트의 처리 순서 (공통)
1. **컨트롤러 탐색:** 스프링 컨테이너 안에 해당 요청을 처리할 수 있는 **Controller**가 있는지 먼저 탐색
2. **정적 리소스 탐색:** 컨트롤러가 없을 경우, `resources/static` 폴더에서 일치하는 정적 컨텐츠(파일)를 찾아 반환

### 2. 정적 컨텐츠 (Static Content)
- **정의:** 서버에서 별도의 처리 없이 파일을 그대로 웹 브라우저에 내려주는 방식
- **위치:** 주로 `resources/static` 디렉토리에 위치
- **특징:** 클라이언트가 요청한 파일 그대로가 응답됨

### 3. MVC와 템플릿 엔진
- **정의:** Model, View, Controller 방식을 사용하여, 서버에서 HTML을 **동적으로** 변환(렌더링)한 뒤 브라우저에 내려주는 방식
- **특징:**
    - 과거의 JSP, PHP와 유사한 방식
    - 정적 컨텐츠와 달리 서버 사이드에서 데이터를 가공하여 화면을 동적으로 변경 가능

### 4. API
- **정의:** HTML(화면)을 내려주는 것이 아니라, 데이터(주로 JSON 포맷)를 클라이언트에게 직접 전달하는 방식
- **활용:**
    - **데이터 전송:** 화면(UI) 없이 데이터만 교환해야 할 때 사용
    - **Modern Web:** React, Vue.js 등 클라이언트 사이드 렌더링(CSR) 프레임워크와 통신 시 사용
    - **App 연동:** 모바일 앱(iOS, Android)과 데이터 통신 시 사용

---

## @ResponseBody

### 1. 정의
- HTTP 통신 프로토콜의 BODY 영역에 **데이터를 직접 반환**하겠다는 의미의 어노테이션
- 뷰 템플릿(View Template)을 거치지 않고, 데이터를 클라이언트에게 바로 전송함

```java
@GetMapping("hello-string")
@ResponseBody   // 데이터 그 자체를 전달하므로 Model이 필요 없음
public String helloString(@RequestParam String name) {
    return "hello " + name;
}
```

```html
<!-- 페이지 소스-->
hello james
```

### 2. 핵심 동작 방식
- **ViewResolver 미사용:** 기존의 MVC 방식과 달리 `viewResolver`가 실행되지 않음
- **HttpMessageConverter 동작:** 대신 적절한 `HttpMessageConverter`가 동작하여 반환 타입을 처리
    - **기본 문자열 처리:** `StringHttpMessageConverter`(StringConverter)가 동작 (단순 문자열 반환)
    - **객체 처리:** `MappingJackson2HttpMessageConverter`(JsonConverter)가 동작 (객체를 **JSON** 스타일로 변환하여 반환)
      - Jackson: 객체 ↔ JSON 변환 라이브러리
      - 즉, `@ResponseBody`가 붙어 있고, 반환 타입이 객체면 스프링이 자동으로 Jackson을 호출하여 처리

### 3. 사용 목적
- **API 개발:** 웹 페이지(HTML)가 아닌, JSON 형식의 데이터를 클라이언트(React, Vue, 모바일 앱 등)에 전달할 때 필수적으로 사용