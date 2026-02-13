## 정적(static) 리소스 vs 템플릿(templates)

### 1. 개념 및 역할 비교
두 폴더는 "서버가 파일을 어떻게 처리해서 주느냐"에 따라 구분된다.

**static (정적 컨텐츠)**
- **경로:** `resources/static/`
- **성격:** 서버가 내용을 건드리지 않고 파일 **그대로** 전달함
- **대상:** HTML(단순), CSS, JS, 이미지, 폰트 등
- **특징:** 프로그래밍 로직이 없음 (누가 접속하든 늘 똑같은 화면)

**templates (동적 컨텐츠)**
- **경로:** `resources/templates/`
- **성격:** 템플릿 엔진(Thymeleaf 등)이 데이터를 가공(바인딩)하여 변환한 뒤 전달함
- **대상:** Thymeleaf, Mustache, JSP 파일 등
- **특징:** 화면에 `[사용자명]` 처럼 동적으로 변하는 데이터가 포함됨

### 2. 동작 원리 (Controller와의 관계)
스프링 부트는 요청이 **Controller를 거치는지 여부**에 따라 파일을 찾는 위치가 다릅니다.

**Case A: URL로 파일 경로를 직접 입력할 때**
- **요청:** `localhost:8080/hello.html`
- **동작:** **Controller를 거치지 않음** -> 정적 리소스 핸들러 작동
- **결과:** `resources/static/hello.html`을 찾아 그대로 보여줌

**Case B: Controller를 통해 화면을 띄울 때**
- **요청:** Controller의 메서드 실행 (`return "hello";`)
- **동작:** **Controller를 거침** -> View Resolver 작동
- **결과:** `resources/templates/hello.html`을 찾아 데이터를 입힌 후 보여줌
- **주의:** Controller가 문자열 반환 시, 스프링은 무조건 `templates` 폴더를 뒤짐 (static에 있으면 404 에러)

### 3. 핵심 요약 (사용 규칙)
- **Controller 사용 O:** 무조건 **`templates`** 폴더에 넣어야 함 (내용이 정적이더라도 Controller를 탄다면 여기 있어야 함)
- **Controller 사용 X:** 무조건 **`static`** 폴더에 넣어야 함 (이미지, CSS, JS, 또는 외부에서 바로 접근할 단순 HTML)

---

## 웹 MVC 동작 원리와 Form 데이터 처리

### 1. input 태그의 속성 비교 (name vs id)
서버 전송 시 가장 중요한 것은 `id`가 아니라 `name`이다.

- **name="username"**
    - **역할:** 서버(Spring)가 데이터를 인식하는 키(Key)값
    - **동작:** 폼 전송 시 `username=입력값` 형태로 서버에 날아감
    - **중요:** DTO(`MemberForm`)의 필드명과 **정확히 일치**해야 데이터가 들어감
- **id="username"**
    - **역할:** 화면(Client)에서 식별하기 위한 값
    - **용도:** CSS 스타일링, 자바스크립트(`getElementById`), 라벨(`label for=`) 연결용
    - **참고:** 서버는 `id` 값에 전혀 관심이 없음

### 2. GET vs POST (데이터 전송 방식의 차이)
데이터를 실어 나르는 위치(Vehicle)가 다르다.

**GET (조회용)**
- **데이터 위치:** **URL 뒤**에 쿼리 스트링으로 붙어서 감
    - 예: `localhost:8080/members?page=1&age=20`
- **특징:** 데이터가 다 보이므로 보안에 취약, 전송 용량 제한 있음
- **용도:** 데이터 변경 없이 **조회**만 할 때 (신문 기사 읽기)

**POST (등록/변경용)**
- **데이터 위치:** HTTP Body(본문) 안에 넣어서 감
    - 예: URL은 `/members/new`로 깔끔하고, 내부에 `name=spring` 데이터가 숨어 있음
- **특징:** 데이터가 URL에 노출되지 않음, 대용량 데이터 전송 가능
- **용도:** 데이터를 **등록, 수정, 삭제**하여 서버의 상태를 바꿀 때 (글쓰기, 회원가입)

### 3. Spring MVC 커맨드 객체 (Command Object)
컨트롤러 메서드(`create(MemberForm form)`)가 파라미터로 **객체**를 받을 때 일어나는 자동화 기술이다.

**용어 정의**
- **커맨드 객체 (Command Object):** `MemberForm`처럼 HTTP 요청 데이터를 담아내기 위해 사용하는 **일반 자바 객체(DTO)**
- **데이터 바인딩 (Data Binding):** 요청 파라미터 값을 객체의 프로퍼티(Setter)에 자동으로 채워주는 과정

**동작 프로세스**
1. **객체 생성:**
    - 스프링은 메서드 파라미터(`MemberForm form`)를 보고, 해당 클래스의 **기본 생성자**를 호출하여 빈(empty) 객체(`new MemberForm()`)를 만듦.
2. **요청 데이터 스캔:**
    - HTML Form이나 URL에서 넘어온 **`key=value`** 데이터(`name=spring`)를 확인.
3. **프로퍼티 매핑 (Setter 호출):**
    - 생성된 객체에서 key 이름(`name`)과 일치하는 Setter 메서드(`setName`)를 찾아 값을 주입함.
    - 즉, 내부적으로 `form.setName("spring")`이 실행됨.
4. **결과:**
    - 데이터가 채워진 객체(`form`)가 `create()` 메서드의 인자로 전달됨.

### 4. `redirect:/` (PRG 패턴)
- **기술적 명칭:** Post-Redirect-Get (PRG) 패턴
- **목적:** POST 요청(데이터 변경) 처리 후, 브라우저를 다른 URL로 리다이렉트(재요청)시켜 중복 처리를 방지함
- **동작 순서:**
    1. **POST:** 클라이언트가 데이터를 전송 (글 등록 요청)
    2. **Redirect:** 서버가 작업 처리 후, 응답 헤더(`Location`)에 이동할 주소를 담아 보냄 (`HTTP 302`)
    3. **GET:** 브라우저가 해당 주소로 자동으로 새로운 GET 요청을 보냄 (조회 화면으로 이동)
- **효과:** 사용자가 [새로고침]을 눌러도 마지막 요청인 GET(조회)만 반복되므로, 데이터가 중복 등록되지 않음

---

## Controller의 Model과 Thymeleaf 문법

### 1. Model 파라미터 (데이터 운반체)
- **정의:** 컨트롤러에서 생성된 **데이터를 화면(View)으로 전달하기 위해 사용하는 컨테이너(객체)**
- **역할:**
    - 비즈니스 로직(Service, Repository)을 수행한 **결과 데이터**를 담는 그릇
    - `model.addAttribute("key", value)` 메서드를 사용하여 데이터를 저장함
- **흐름:** Controller (데이터 담기) -> Model (운반) -> View (데이터 꺼내기)

### 2. Thymeleaf의 `${...}` 문법
- **정의:** **변수 표현식 (Variable Expressions)**
- **동작:** Controller가 `Model`에 담아서 넘겨준 데이터를 **키(Key) 값으로 조회**하여 화면에 출력하는 문법
- **예시:**
    - **Controller:** `model.addAttribute("members", members);` (키: members)
    - **HTML:** `<tr th:each="member : ${members}">` (모델에 담긴 members 리스트를 꺼내서 반복함)
