## 특정 프로젝트를 Gradle 프로젝트로 인식시키기
**문제 상황:** 기존 프로젝트를 열었는데, Gradle Project로 인식하지 못하는 경우

**해결 방법:**
1. 해당 프로젝트에 위치한 `build.gradle` 파일 우클릭
2. `Link Gradle Project(Import)` 클릭

---

## 템플릿 엔진
### 1. 정의
- 템플릿 양식(Template)과 데이터 모델(Data Model)을 결합하여 결과 문서(Result Document)를 생성하는 소프트웨어 혹은 라이브러리
- 웹 개발 환경에서는 주로 **서버 사이드에서 데이터를 포함한 HTML 문서를 동적으로 생성**하여 클라이언트에 응답하는 역할 수행

### 2. 핵심 구성 요소
1. **템플릿 (Template):** 말그대로 문서 양식을 뜻함. 데이터가 삽입될 위치는 표현식이나 태그로 정의되어 있음 (예: `.html`, `.jsp`)
2. **데이터 모델 (Data Model):** 결과 데이터를 담는 모델
3. **프로세서 (Processor):** 템플릿 파일을 파싱(Parsing)하고, 정의된 문법에 따라 데이터를 바인딩(Binding)하여 최종 결과물을 생성하는 엔진 본체

### 3. 작동 원리 (Server-Side Rendering 기준)
스프링 MVC에서의 일반적인 흐름은 다음과 같다.
1. **요청(Request):** 클라이언트가 웹 브라우저를 통해 특정 페이지를 요청한다.
2. **데이터 처리:** 컨트롤러가 비즈니스 로직을 수행하고 결과 데이터를 Model 객체에 담는다.
3. **뷰 선택:** 컨트롤러가 뷰 이름(View Name)을 반환하면, ViewResolver가 해당 이름에 매핑되는 템플릿 파일을 찾는다.
4. **렌더링(Rendering):** 템플릿 엔진(프로세서)이 템플릿 파일의 태그와 표현식을 해석하고, Model에 담긴 데이터를 해당 위치에 치환하거나 제어문(반복, 조건 등)을 실행하여 순수 HTML을 생성한다.
5. **응답(Response):** 생성된 HTML 코드가 HTTP 응답 본문(Body)에 담겨 클라이언트로 전송된다.

### 4. 코드 예시 (Thymeleaf)
**A. 템플릿 파일 (HTML + Thymeleaf 문법)**

정적 마크업 내에 `th:text`와 같은 속성을 사용하여 동적 데이터 처리를 선언한다.
```html
<div>
    <p th:text="${data}">기본 텍스트</p>
</div>
```

**B. 데이터 모델 (Java Controller)**

```java
model.addAttribute("data", "Hello Spring");
```

**C. 렌더링 결과 (HTML)**

템플릿 엔진 처리 후 브라우저가 수신하는 최종 코드.
```html
<div>
    <p>Hello Spring</p>
</div>
```

### 5. 주요 특징 및 이점
* **재사용성:** 헤더(Header) 등 반복되는 레이아웃을 모듈화하여 여러 페이지에서 재사용할 수 있다.
* **유지보수성:** 디자인(HTML/CSS)과 로직(Java Code)이 분리되어 있어, 코드가 섞여 있는 방식보다 유지보수가 용이하다.
* **서버 사이드 렌더링 (SSR):** 서버에서 완전한 HTML을 만들어 보내므로 SEO(검색 엔진 최적화)에 유리하다.

### 6. 종류
* **Thymeleaf:**
  * 스프링 부트에서 권장하는 엔진. 
  * 순수 HTML 구조를 유지하면서 속성을 통해 동작하는 것이 특징(Natural Template).
* **JSP (JavaServer Pages):** 
  * 과거에 주로 사용되던 방식. 
  * HTML 내에 자바 코드를 직접 삽입할 수 있으나 현재는 권장되지 않음.

---

## Gradle 프로젝트 핵심 파일 가이드

### 1. build.gradle

**정의 및 역할**

- 프로젝트의 빌드 구성 정보(Build Configuration)를 정의하는 가장 핵심적인 스크립트 파일
- 가장 많이 수정하게 될 파일로, **프로젝트에 필요한 라이브러리나 플러그인, 빌드 작업 등을 설정**한다.

**주요 포함 내용**

* **Plugins:** 프로젝트에 적용할 플러그인 (예: `java`, `org.springframework.boot`).
* **Dependencies:** 프로젝트에서 사용할 외부 라이브러리 (예: `spring-boot-starter-web`, `h2`, `lombok`).
* **Repositories:** 라이브러리를 다운로드할 저장소 위치 (예: `mavenCentral()`).
* **Task:** 빌드 과정에서 수행할 구체적인 작업 정의.

### 2. settings.gradle

**정의 및 역할**

- 프로젝트의 구성 정보(Project Settings)를 관리하는 설정 파일
- 빌드 프로세스가 시작될 때 가장 먼저 실행되며, **어떤 프로젝트들이 빌드에 포함되어야 하는지를 정의**한다.

**주요 포함 내용**

* **rootProject.name:** 최상위 프로젝트의 이름 설정.
* **include:** 멀티 모듈 프로젝트인 경우, 하위 모듈(서브 프로젝트)을 빌드에 포함시킬 때 사용.

### 3. gradlew & gradlew.bat

**정의 및 역할**

- **Gradle Wrapper**를 실행하기 위한 스크립트 파일
- 로컬 컴퓨터에 Gradle을 별도로 설치하지 않아도, 프로젝트 내에 포함된 이 스크립트를 통해 특정 버전의 Gradle을 사용하여 빌드를 수행할 수 있게 해준다.

**종류**

- **A. gradlew:** Linux 또는 macOS 환경에서 사용하는 쉘 스크립트 파일
- **B. gradlew.bat:** Windows 환경에서 사용하는 배치(Batch) 스크립트 파일

## 동작 구조 (내장 톰캣 & 스프링 컨테이너)

### 1. 요청 수신 (내장 톰캣)
* **역할:** 클라이언트의 HTTP 요청을 받아 스프링 컨테이너로 전달.
* **Flow:** `웹 브라우저` -> `내장 톰캣` -> `스프링 컨테이너`
* *(예시: `localhost:8080/hello` 요청 수신)*

### 2. 스프링 컨테이너 (Spring Container)
핵심 컴포넌트들이 빈(Bean)으로 관리되는 공간.

#### A. 컨트롤러 탐색 및 실행
* 요청된 URL에 매핑된 **Controller**를 찾아 실행한다.
* *(예시: `@GetMapping("hello")`가 붙은 `helloController` 실행)*

#### B. 비즈니스 로직 및 반환
* 로직 수행 후, 화면에 보여줄 데이터(**Model**)와 화면의 이름(**ViewName**)을 반환한다.
* *(예시: `model.addAttribute("data", "hello!")` 후 `return "hello"`)*

#### C. 뷰 리졸버 (ViewResolver) 동작
* 컨트롤러가 반환한 ViewName을 가지고 실제 템플릿 파일을 탐색한다.
* 스프링 부트 기본 설정: `resources/templates/` + `{ViewName}` + `.html`
* *(예시: `resources/templates/hello.html` 탐색)*

#### D. 템플릿 엔진 처리 (Thymeleaf)
* 템플릿 파일에 Model 데이터를 입혀 변환(렌더링)한 후, 최종 HTML을 톰캣 서버로 넘긴다.

## 터미널 환경 비교 및 Git Bash 사용 이유

### 1. PowerShell vs Linux (Bash) 차이

**A. 기반 철학의 차이**
* **PowerShell (Windows):** 
  * **객체(Object)** 중심. 
  * `.NET` 프레임워크 기반으로 작동하며, 명령어의 결과가 단순 텍스트가 아닌 객체로 반환된다.
* **Linux Bash:** 
  * **텍스트(Text)** 중심. 
  * 모든 데이터가 문자열(String) 흐름으로 처리된다. (서버 개발의 표준)

**B. 명령어와 호환성**
* **PowerShell:** 
  * `ls`, `rm` 같은 명령어가 존재하지만, 이는 별칭(Alias)일 뿐이다. 
  * 실제로는 `Get-ChildItem` 같은 윈도우 고유 명령어가 실행되므로, 리눅스 옵션(`-arlth` 등)이 동작하지 않는다.
* **Linux Bash:** 
  * 고유의 리눅스 명령어(`ls`, `rm`, `grep`)와 옵션을 그대로 사용한다.

### 2. 왜 Git Bash를 사용하는가?

**A. 강의 및 실무 환경과의 통일**
* 대부분의 서버 개발 강의나 문서는 **Mac/Linux**를 기준으로 설명한다.
* Git Bash를 사용하면 윈도우에서도 리눅스 명령어(Bash)를 에뮬레이션(모방)하여 사용할 수 있어, 별도의 명령어 변환 없이 강의를 그대로 따라갈 수 있다.

**B. 개발 편의성**
* 리눅스 기반으로 동작하는 서버 배포 환경과 유사한 환경을 로컬(윈도우)에서 경험할 수 있다.

### 3. 인텔리제이의 기본 터미널을 Git Bash로 바꾸는 법

Settings → Terminal → Shell Path를 bash로 변경한다. (`-login -i` 옵션 추가)