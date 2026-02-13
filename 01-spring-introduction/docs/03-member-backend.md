## 서비스(Service) 계층
### 1. 정의
- 도메인 객체를 활용하여 **핵심 비즈니스 로직**이 동작하도록 구현하는 계층
- 리포지토리가 단순히 데이터를 넣고 빼는 기계적인 역할이라면, 서비스는 비즈니스 의존적인 용어(회원가입, 중복확인 등)를 사용하여 로직을 설계함

## 실무에서의 동시성 처리 (Concurrency)
> **Note:** 예제에서는 학습의 단순화를 위해 기본 자료형을 사용하지만, 실무(멀티 쓰레드 환경)에서는 동시성 이슈를 반드시 고려해야 함.

### 1. 데이터 저장소 (Map)
- **예제:** `HashMap` (Thread-safe하지 않음)
- **실무:** `ConcurrentHashMap` 사용 (여러 쓰레드가 동시에 접근할 때 발생하는 동시성 문제 방지)

### 2. ID 생성 (Sequence)
- **예제:** `long`
- **실무:** `AtomicLong` 사용 (동시 접근 시 ID 값이 꼬이거나 중복되는 문제 방지)

## Optional 활용
### 1. Optional.ofNullable()
- **역할:** 반환 값이 `null`일 가능성이 있는 경우 사용
- **장점:** 결과값을 감싸서 반환하면, 클라이언트(호출하는 쪽)에서 `null` 체크 로직(`if (result == null)`) 대신 `ifPresent` 등을 통해 더 유연하고 안전하게 흐름을 제어할 수 있음

## Java 8 Stream API

### 1. 정의
- **데이터의 흐름(Stream):** 컬렉션(List, Map 등)이나 배열에 저장된 요소들을 하나씩 참조하여 **반복적인 처리**를 가능하게 하는 기능 (Java 8부터 도입)
- **선언형 코드:** "어떻게(How)" 구현할지보다 "무엇을(What)" 할지에 집중하여 코드가 간결하고 가독성이 높아짐

### 2. 코드 분석 (`findByName`)
아래 코드는 `store`에 있는 모든 멤버 중 이름이 같은 사람을 찾아내는 로직이다.
```java
private static Map<Long, Member> store = new HashMap<>();

@Override
public Optional<Member> findByName(String name) {
    return store.values().stream()  // 1. 스트림 생성 (데이터 흐름 시작)
            .filter(member -> member.getName().equals(name))    // 2. 필터 (조건에 맞는 요소만 거르기)
            .findAny(); // 3. 최종 연산
}
```

1. **`stream()`**
    - 컬렉션(여기서는 `store.values()`)을 스트림으로 변환하여 파이프라인(작업 흐름)을 만듦
    - 마치 공장의 컨베이어 벨트에 데이터를 올리는 과정

2. **`filter(Predicate)`**
    - **역할:** 말그대로 필터 역할, 즉 조건에 맞지 않는 요소를 걸러내는 거름망 역할
    - **동작:** `member` 객체를 하나씩 꺼내서 이름이 `name`과 같은지 확인(`equals`)하고, `true`인 요소만 다음 단계로 통과시킴

3. **`findAny()`**
    - **역할:** 조건(filter)을 통과한 요소 중 **아무거나 하나**를 찾으면 즉시 반환
    - **반환 타입:** 찾을 수도 있고 없을 수도 있으므로 `Optional`로 감싸서 반환
    - **참고:** 순서가 중요한 스트림에서는 주로 `findFirst()`를 쓰지만, 병렬 처리 등 순서가 상관없을 때는 `findAny()`가 더 빠름 (결과적으로 하나만 찾으면 되기 때문)

---

## 테스트 케이스 작성 기초

### 1. 테스트 클래스와 Optional
- **접근 제어자:** 테스트 클래스는 외부에서 호출할 일이 없으므로 `public`을 생략해도 무방함 (JUnit 5부터 가능)
- **Optional 값 꺼내기:**
    - 실제 코드에서는 안전한 처리가 필요하지만, 테스트 코드에서는 검증이 목적이므로 `get()`을 사용해 바로 값을 꺼내도 됨
    - 만약 값이 없으면 예외가 발생하여 테스트가 실패하므로, 그것 자체로 검증이 됨

### 2. 검증(Assertions) 라이브러리 비교 (JUnit vs AssertJ)
- **JUnit (`org.junit.jupiter.api`):**
    - 자바 표준 테스트 프레임워크인 JUnit이 제공하는 기본 검증 기능
    - 문법: `Assertions.assertEquals(expected, actual)`
    - 단점: 기대값과 실제값의 인자 순서가 헷갈릴 수 있음
- **AssertJ (`org.assertj.core.api`):**
    - 테스트 검증을 위해 만들어진 더 강력하고 직관적인 라이브러리 (스프링 부트 기본 포함)
    - 문법: `assertThat(actual).isEqualTo(expected)`
    - 장점:
        - "이 값은(actual) 저 값과(expected) 같아야 한다"처럼 문장을 읽듯 자연스러움
        - 메서드 체이닝을 지원하여 다양한 검증을 이어서 작성 가능
    - **결론:** 실무에서는 가독성이 월등히 좋은 **AssertJ**를 주로 사용함

### 3. Static Import
- **정의:** 다른 클래스의 정적(static) 메서드를 클래스명 없이 바로 사용할 수 있게 해주는 자바 문법
- **효과:** 코드가 훨씬 간결해지고 가독성이 높아짐
    - **사용 전:** `Assertions.assertThat(member).isEqualTo(result);`
    - **사용 후:** `assertThat(member).isEqualTo(result);`
- **방법:** (IntelliJ 기준) `Assertions`에 커서를 두고 `Alt` + `Enter` → `Add on-demand static import...` 선택

### 4. 테스트 생명주기와 격리성
- **순서 비보장:** 모든 테스트 메서드는 실행 순서가 보장되지 않음
- **문제점:** 먼저 실행된 테스트가 저장소에 남긴 데이터 때문에 다음 테스트가 실패할 수 있음
- **해결책 (`@AfterEach`):**
    - 각 테스트 메서드가 끝날 때마다 무조건 실행되는 콜백 메서드
    - 주로 저장소(Map 등)를 비우는(`clearStore`) 코드를 작성하여, 테스트가 서로 독립적으로 실행되도록 보장함

---

## Given-When-Then 패턴

### 1. 정의
- **BDD(Behavior Driven Development)** 스타일의 테스트 코드를 작성하기 위한 대표적인 패턴
- 테스트 코드를 준비(Given), 실행(When), 검증(Then)의 3단계로 명확하게 나누어 작성하는 방식

### 2. 단계별 설명
1. **Given (준비):** 테스트를 수행하기 위한 **기반 데이터**나 **상황**을 세팅하는 단계 (변수 선언, 객체 생성 등)
2. **When (실행):** 실제 검증하고자 하는 핵심 기능(메서드)을 실행하는 단계
3. **Then (검증):** 실행 결과가 기대한 값과 일치하는지 확인(Assertions)하는 단계

### 3. 코드 예시
```java
@Test
void save() {
    // Given (이런 데이터가 주어졌을 때)
    Member member = new Member();
    member.setName("spring");

    // When (이 함수를 실행하면)
    repository.save(member);

    // Then (결과는 이래야 한다)
    Member result = repository.findById(member.getId()).get();
    assertThat(result).isEqualTo(member);
}
```

---

## 회원 서비스 개발 (Service Layer)

### 1. IntelliJ 필수 리팩토링 단축키
개발 생산성을 높여주는 핵심 단축키이다. (Windows/Linux 기준)

**변수 추출하기 (Extract Variable)**
- **단축키:** `Ctrl` + `Alt` + `V`
- **기능:** 메서드 호출부 뒤에서 실행하면, 반환 타입에 맞는 **변수 선언과 할당 코드를 자동으로 생성**해줌
- **예시:** `repository.findById(id)` 뒤에서 입력 시 `Optional<Member> member = ...` 자동 완성

**메서드 추출하기 (Extract Method)**
- **단축키:** `Ctrl` + `Alt` + `M`
- **기능:** 드래그한 코드 블록을 **새로운 메서드로 분리**해줌
- **사용 시점:** 메서드가 너무 길거나, **특정 로직(예: 중복 회원 검증)에 이름을 붙여** 가독성을 높일 때

### 2. Optional 활용 (`orElseGet`)
- **기능:** Optional 객체 안에 값이 있으면 가져오고, **값이 없으면(null이면) 인자로 전달된 메서드를 실행**하여 값을 반환함
- **특징:** `get()`으로 무작정 꺼내는 것보다 안전하며, 값이 없을 때만 로직이 실행되므로 효율적임
- **비교:** `orElseThrow`는 값이 없을 때 예외를 발생시킴 (회원 가입 중복 검증 로직 등에서 사용)

### 3. 리포지토리 vs 서비스 (계층별 네이밍 규칙)
각 계층은 역할에 맞는 용어를 사용하여 의도를 명확히 드러내야 한다.

**리포지토리 (Repository)**
- **성격:** 기계적, 데이터베이스 친화적
- **용어:** `save`, `findById`, `delete` 등
- **역할:** 단순히 데이터를 넣고 빼는 **영속 계층(Persistence Layer)**

**서비스 (Service)**
- **성격:** 비즈니스 의존적, 기획자/운영자와 소통 가능한 용어
- **용어:** `join`(가입), `findMembers`(전체 회원 조회) 등
- **역할:** 비즈니스 도메인 객체를 가지고 **핵심 비즈니스 로직**을 수행하는 계층

---

## 계층 간 데이터 전달 (DTO 패턴)

### 1. DTO (Data Transfer Object)
**정의 및 역할**
- **정의:** 계층(Layer) 간 데이터 교환을 위해 사용하는, 로직을 갖지 않는 순수한 데이터 객체
- **핵심 용도:** 주로 **클라이언트(Web)와 컨트롤러(Controller) 사이**에서 요청과 응답 데이터를 주고받을 때 사용
- **필요성:** 내부 도메인 객체(Entity/Member)를 외부(화면, API)에 직접 노출하지 않기 위함
- **예시:** `MemberForm`, `MemberRequest`, `MemberResponse`

### 2. Controller와 Service의 동작 흐름
실무에서 데이터가 전달되는 표준적인 과정이다.

**Controller (요청 수신 및 변환)**
- **입력:** 웹 브라우저의 요청 데이터를 **DTO**(`MemberForm`) 형태로 받음
- **변환:** 받은 DTO 데이터를 실제 비즈니스 로직 처리를 위해 **도메인 객체**(`Member`)로 변환함 (단, 복잡한 경우 서비스 계층으로 DTO를 그대로 넘기기도 함)
- **호출:** 변환된 객체를 인자로 담아 Service 계층을 호출함

**Service (비즈니스 로직 수행)**
- **입력:** Controller로부터 **도메인 객체**(`Member`)를 넘겨받음
- **수행:** 핵심 비즈니스 로직(`join` 등)을 수행하고 Repository를 통해 DB에 저장

---

## 회원 서비스 테스트 및 DI(의존성 주입) 기초

### 1. 테스트 코드 작성 팁
**IntelliJ 단축키**
- **단축키:** `Ctrl` + `Shift` + `T`
- **기능:** 현재 클래스에 대응하는 **테스트 클래스(껍데기)를 자동으로 생성**해줌 (패키지 구조까지 맞춰줌)

**메서드 명명 규칙**
- **특징:** 테스트 메서드 이름은 과감하게 **한글로 작성해도 무방함**
- **이유:** 테스트 코드는 빌드될 때 실제 배포 코드에 포함되지 않으며, 직관적으로 어떤 기능을 검증하는지 알아보는 것이 더 중요하기 때문

**검증의 중요성**
- **원칙:** 정상적으로 작동하는 것보다, **예외 상황(Exception Flow)이 터져야 할 때 잘 터지는지 검증**하는 것이 훨씬 중요함

### 2. Static과 인스턴스의 차이
**Static 변수의 특징**
- **정의:** `static` 키워드가 붙은 변수는 인스턴스(객체) 레벨이 아니라 **클래스 레벨**에 붙음
- **현상:** `new MemoryMemberRepository()`로 서로 다른 객체 2개를 생성하더라도, 내부의 `static Map`은 클래스 레벨에서 하나만 존재하므로 데이터를 공유하게 됨 (그래서 테스트가 통과하는 것처럼 보임)

### 3. DI (Dependency Injection) 도입 배경
**문제점**
- 서비스 코드 내부에서 리포지토리를 직접 `new`로 생성하고, 테스트 코드에서도 `new`로 생성하면 **서로 다른 리포지토리 인스턴스**를 사용하는 것임
- `static`을 빼면 바로 데이터 공유가 안 되는 문제가 발생함

**해결책: 의존성 주입 (DI)**
- **정의:** 객체를 직접 생성(new)하지 않고, **외부에서 생성된 객체를 넣어주는(주입받는) 방식**
- **구현:** `MemberService`의 생성자(Constructor)를 통해 외부에서 `repository`를 받도록 코드를 변경함
- **효과:** 테스트 코드와 서비스 코드가 **같은 리포지토리 인스턴스**를 바라보게 되어, 데이터 동기화 문제 해결 및 테스트의 정확성 보장

---

## 회원 서비스 테스트 심화 (Test)

### 1. 예외 처리 전략 (Exception)
중복 회원 검증 시 어떤 예외를 사용할지에 대한 관점 차이이다.

**IllegalStateException (강의 선택)**
- **의미:** 메서드 호출 인자는 정상이나, 시스템의 현재 상태(DB에 이미 회원이 존재함)로 인해 처리가 불가능함
- **논리:** "이름 자체는 문제없지만, 이미 가입된 상태라 처리할 수 없어."

**IllegalArgumentException**
- **의미:** 메서드에 전달된 **인자(Argument)** 자체가 부적절함
- **논리:** "이미 존재하는 이름을 입력값으로 넘겼으므로, 이 입력은 잘못되었어."
- **결론:** 둘 다 실무에서 사용 가능하며, 상황과 팀의 컨벤션에 따라 선택

### 2. 테스트와 Static 변수의 관계
`@BeforeEach`에서 객체를 새로 생성(`new`)하더라도 `@AfterEach`가 반드시 필요한 이유이다.

**Static 변수의 함정**
- **원인:** `MemoryMemberRepository`의 `store` 변수가 `static`으로 선언되어 있음
- **동작:** `static` 필드는 객체(Instance)가 아닌 **클래스(Class) 레벨**에 존재함. 즉, `new`로 리포지토리 객체를 100개 만들어도 **모든 객체가 단 하나의 `store`를 공유**하게 됨
- **결론:** 테스트 간 데이터 격리를 위해, 매 테스트가 끝날 때마다 `@AfterEach`에서 `store.clear()`를 호출하여 공용 저장소를 비워줘야 함

### 3. Given-When-Then 패턴의 위치 선정
테스트의 목적(검증 대상)에 따라 코드가 위치할 절(Given/When)이 달라진다.

**예시: 중복 회원 예외 테스트**
- **목적:** "중복된 회원을 가입시킬 때(When) 예외가 터지는가?"
- **전제 조건(Given):** 이미 첫 번째 회원이 가입되어 있어야 함 (`memberService.join(member1)`)
- **실행(When):** 두 번째 회원을 가입 시도함 (`memberService.join(member2)`)
- **팁:** 테스트의 핵심 실행 동작을 제외한 모든 '세팅' 과정은 `Given`에 위치하는 것이 논리적임

---

## 테스트 검증 용어 (Actual vs Expected)

### 1. 정의
테스트 코드는 "내가 의도한 값"과 "실제 나온 값"을 비교하는 과정이다.

**Actual (실제값)**
- **정의:** 테스트 대상 메서드를 실행해서 나온 **진짜 결과값**
- **위치:** `result`, `member` 등 로직 수행 후 반환된 변수
- **의미:** "코드 돌려보니까 실제로 이게 나왔어."

**Expected (기대값)**
- **정의:** 테스트를 통과하기 위해 나와야 한다고 **내가 정해둔 정답**
- **위치:** "hello", `100`, `true` 등 개발자가 직접 입력하거나 미리 준비해둔 값
- **의미:** "이게 나와야 정상이야."

### 2. 라이브러리별 파라미터 순서 (중요)
검증 실패 시 출력되는 에러 메시지("Expected A but was B")의 정확성을 위해 순서를 지키는 것이 중요하다.

**JUnit 5 (`Assertions`)**
- **문법:** `assertEquals(expected, actual)`
- **순서:** 기대값(정답)이 먼저 나옴
- **단점:** 순서가 헷갈려서 반대로 넣으면, 에러 로그가 "기대값은 5인데 실제로는 3이 나왔어"라고 해야 할 것을 "기대값은 3인데 실제로는 5가 나왔어"라고 거꾸로 말해서 혼란을 줌

**AssertJ (`assertThat`) - 추천**
- **문법:** `assertThat(actual).isEqualTo(expected)`
- **순서:** 실제값(결과)이 먼저 나옴
- **장점:** "이 결과값(actual)은 저 정답(expected)과 같아야 해"라고 영어 문장처럼 읽혀서 순서 혼동이 적음

---

## assertThrows (JUnit 5)

### 1. 정의
- **역할:** 특정 로직 실행 시, 기대하는 예외(Exception)가 실제로 발생하는지 검증하는 메서드
- **결과:**
    - 예외가 발생하면? -> **테스트 통과 (Success)**
    - 예외가 발생하지 않거나 다른 예외가 발생하면? -> **테스트 실패 (Fail)**

### 2. 문법 구조
```java
// 반환값(Exception) = assertThrows(기대하는_예외_클래스, 실행할_로직_람다);
IllegalStateException e = assertThrows(IllegalStateException.class,
    () -> memberService.join(member2));
```

### 3. 주요 활용 (반환값 검증)
`assertThrows`는 발생한 **예외 객체 자체를 반환**하므로, 이를 받아서 **에러 메시지**까지 정확한지 추가로 검증할 수 있다.

```java
// 1. 예외 발생 여부 검증
IllegalStateException e = assertThrows(IllegalStateException.class, 
    () -> memberService.join(member2));

// 2. 에러 메시지 내용 검증 (Actual vs Expected)
assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
```