## 스프링 AOP (Aspect Oriented Programming)

### 1. 개요 및 필요성
* **핵심 관심 사항 (Core Concern):** 비즈니스 로직 (예: 회원 가입, 주문, 결제 등).
* **공통 관심 사항 (Cross-cutting Concern):** 애플리케이션 전반에 걸쳐 공통적으로 적용되는 부가 기능 (예: 로깅, 시간 측정, 보안, 트랜잭션).
* **문제점:** 공통 로직을 모든 핵심 로직에 일일이 복사-붙여넣기 하면 코드가 지저분해지고 유지보수가 불가능해짐.
* **해결책:** AOP를 통해 공통 관심 사항을 **따로 떼어내서 관리**하고, 원하는 곳에만 **적용**함.

---

### 2. AOP 등록 방식: 자동 vs 수동
AOP는 클래스에 직접 코드를 넣는 것이 아니라, 런타임에 위빙(Weaving)되므로 가독성이 매우 중요하다.

#### A. 자동 등록 (@Component + @Aspect)
* **방식:** 컴포넌트 스캔을 통해 자동으로 빈으로 등록.
* **단점:** 
  * AOP 클래스가 어디 있는지 찾기 힘들 수 있음. 
  * 개발자가 코드를 볼 때 AOP가 적용되었는지 인지하지 못할 위험이 있음.

#### B. 수동 등록 (@Bean in Config) - **권장**
* **방식:** `SpringConfig` 파일에 직접 `@Bean`으로 등록.
* **장점:**
    * 프로젝트 설정 파일(`SpringConfig`)만 열어보면 **"아! 이 프로젝트에는 AOP가 적용되어 있구나"** 라고 바로 인지 가능.
    * 유지보수성을 위해 AOP 같은 특수 기능은 명시적으로 등록하는 것이 좋음.

---

### 3. 프록시(Proxy) 동작 원리

#### 흐름 (Proxy Pattern)
스프링은 AOP가 적용된 빈을 등록할 때, **진짜 객체 대신 프록시(가짜) 객체**를 컨테이너에 등록한다.

1.  **컨테이너 시작:** `MemberService`에 AOP가 걸려있음을 감지.
2.  **프록시 생성:** `MemberService`를 복제 및 조작한 **프록시 객체(CGLIB)** 생성.
3.  **빈 등록:** 컨테이너에 진짜 대신 **프록시 객체**를 등록.
4.  **실행:**
    * `Controller` -> **Proxy.join()** (공통 로직 실행)
    * `Proxy` -> `joinPoint.proceed()`
    * `RealService` -> **Real.join()** (핵심 로직 실행)

#### CGLIB (Code Generator Library)
* **확인:** `memberService.getClass()` 출력 시 `class com.hello...MemberService$$EnhancerBySpringCGLIB$$...` 확인 가능.
* **기술:** 바이트코드를 조작하여 클래스를 상속받은 가짜 프록시 객체를 생성하는 라이브러리.

---

### 4. IoC와 DI의 역할

#### 질문
> "프록시가 중간에 끼어드는데, 왜 클라이언트(Controller) 코드는 한 줄도 안 고쳐도 될까?"

#### 답변: 제어의 역전 (IoC) 덕분이다.
1.  **DI (의존성 주입):** 컨트롤러는 **"MemberService 타입의 객체면 뭐든 줘"** 라고 선언만 한다. (`private final MemberService service;`)
2.  **IoC (제어의 역전):** 어떤 객체(진짜 vs 프록시)를 주입할지 결정하는 **제어권**은 개발자가 아니라 **스프링 컨테이너**에게 있다.
3.  **결과:** 컨테이너가 AOP 설정을 보고 **알아서 프록시 객체를 주입**해 준다.
    * 개발자는 프록시의 존재를 몰라도 되며, 이는 OCP(개방-폐쇄 원칙)를 완벽하게 지키는 설계이다.

---

### 5. 구현 상세 (JoinPoint)

```java
// 공통 관심 사항을 적용할 타겟 정의
// (hello.hellospring 패키지 하위의 모든 메서드에 적용)
@Around("execution(* hello.hellospring..*(..))")
public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
    // 1. 공통 로직 시작 (예: 시간 측정 시작)
    long start = System.currentTimeMillis();

    try {
        // 2. 진짜 로직 실행 (핵심)
        // 이 코드가 없으면 진짜 메서드는 실행되지 않음
        return joinPoint.proceed(); 
    } finally {
        // 3. 공통 로직 종료 (예: 시간 측정 종료)
        long finish = System.currentTimeMillis();
        System.out.println("END: " + joinPoint.toString() + " " + (finish - start) + "ms");
    }
}
```

* **`@Around`:** 메서드 실행 전후, 예외 발생 시점 등 모든 시점을 제어할 수 있는 가장 강력한 어드바이스.
* **`ProceedingJoinPoint`:** 프록시 내부에서 진짜 객체의 메서드를 호출할 수 있게 해주는 객체.

### 6. AOP 적용 방식 종류
1.  **스프링 AOP (RTW - RunTime Weaving):**
    * **방식:** 프록시(Proxy) 기반.
    * **특징:** 스프링 컨테이너가 뜰 때 가짜 객체를 생성해서 연결함. (프록시를 이용해 실행 시점에 적용)
    * **장점:** 특별한 컴파일러가 필요 없고 설정이 쉬움.
2.  **AspectJ (CTW - CompileTime Weaving):**
    * **방식:** 진짜 자바 코드를 컴파일(.class)할 때, 공통 로직 코드를 물리적으로 **끼워 넣음(Weaving)**.
    * **특징:** 프록시가 아니라 진짜 코드 안에 로직이 박힘. (AspectJ 컴파일러를 이용해 컴파일 시점에 적용)
    * **장점:** 성능이 더 좋고, 메서드 호출뿐만 아니라 필드 접근 등 더 강력한 기능 제공.
    * **단점:** 설정이 복잡함. (실무에서는 대부분 스프링 AOP로 해결 가능)

---

## AOP 핵심 용어 정리

### 1. 주요 용어 정의 및 코드 매핑

| 용어 | 의미                                       | 코드 매핑 (TimeTraceAop 예시) |
| :--- |:-----------------------------------------| :--- |
| **Aspect** | 공통 관심 사항(로직 + 적용 지점)을 하나로 모듈화한 것.        | `TimeTraceAop` **클래스 전체** |
| **Advice** | 실질적으로 **"어떤 일(공통 로직)"** 을 해야 하는지 담은 구현체. | `execute()` **메서드 내부 로직** |
| **Pointcut** | **"어디에 적용할지"** 필터링하는 표현식(Expression).    | `@Around("execution(...)")` |
| **JoinPoint** | AOP가 적용될 수 있는 **모든 실행 시점** (메서드 실행 등).   | 메서드 파라미터 `ProceedingJoinPoint` |
| **Target** | AOP가 적용되는 **실제 객체** (핵심 비즈니스 로직).        | `MemberService` (프록시 내부의 실제 인스턴스) |


### 2. 코드 구조로 보는 AOP

```java
@Aspect // 1. Aspect: 공통 관심 사항의 모듈
@Component // 컴포넌트 스캔을 써도 되지만, 설정 파일에 직접 등록하는 것을 권장
public class TimeTraceAop {

    // 2. Pointcut: 적용 대상 지정 (hello.hellospring 패키지 하위)
    @Around("execution(* hello.hellospring..*(..))")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable { // 3. JoinPoint: 현재 실행 지점 정보
        
        // 4. Advice (Before): 메서드 실행 전 로직
        long start = System.currentTimeMillis();

        try {
            // 5. Target 호출: 실제 객체의 메서드 실행
            return joinPoint.proceed(); 
        } finally {
            // 4. Advice (After): 메서드 실행 후 로직
            long finish = System.currentTimeMillis();
            System.out.println("END: " + joinPoint.toString() + " " + (finish - start) + "ms");
        }
    }
}
```

### 3. Intercept 개념 및 구분

#### 개념적 의미
* **Intercept:** 실행 흐름을 중간에 **가로채는 행위**를 의미함.
* **AOP 동작:** 프록시(Proxy)가 클라이언트의 요청을 **가로채서(Intercept)** 공통 로직(Advice)을 실행하고, 다시 실제 객체(Target)에게 위임하는 방식.

#### 기술적 구분: AOP vs Spring Interceptor

| 구분 | AOP (Aspect Oriented Programming) | 스프링 인터셉터 (Spring Interceptor) |
| :--- | :--- | :--- |
| **적용 단위** | **메서드 (Method)** 단위 | **HTTP 요청 (Request/URL)** 단위 |
| **주요 대상** | Service, Repository 등 비즈니스 계층 | Controller 앞단의 웹 계층 |
| **구현 기술** | 프록시(Proxy) 기반 | `HandlerInterceptor` 인터페이스 구현 |

---

## AOP 순환 참조 문제 (Circular Reference)

### 1. 문제 상황
AOP를 `@Component`가 아닌 **`@Bean`으로 수동 등록**할 때, `SpringConfig` 설정 파일까지 AOP 적용 대상에 포함하면 **순환 참조 오류**가 발생한다.

* **오류 메시지:** 
  * `The dependencies of some of the beans in the application context form a cycle`
  * 즉, 스프링 컨테이너(Application Context)의 일부 빈들의 의존성이 사이클을 형성한다.

### 2. 발생 원인 (메커니즘)
`TimeTraceAop`의 적용 범위(`Pointcut`)가 너무 광범위하게 설정되어, **자기 자신을 생성하는 설정 파일(`SpringConfig`)까지 감시**하려 했기 때문이다.

#### 상세 흐름
1.  **빈 생성 시도:** 스프링 컨테이너가 `TimeTraceAop` 빈을 생성하기 위해 `SpringConfig.timeTraceAop()` 메서드를 호출한다.
2.  **AOP 개입:** 이때, AOP 설정(`execution(* hello.hellospring..*)`)에 의해 `SpringConfig`의 메서드도 감시 대상으로 인식된다.
3.  **모순 발생:**
    * `SpringConfig.timeTraceAop()`를 실행하려면, 먼저 **AOP(TimeTraceAop)** 가 적용되어야 한다.
    * 하지만 **AOP(TimeTraceAop)** 는 지금 막 생성하려는 중이다 (아직 없음).
4.  **결과:** **"AOP를 만들려면 AOP가 필요하다"** 는 무한 루프(Cycle)에 빠져 에러가 발생한다.

### 3. 해결 방법
AOP 적용 대상(`Pointcut`)에서 **설정 파일(`SpringConfig`)을 제외**시켜야 한다.

```java
@Aspect
public class TimeTraceAop {

    // !target(...)을 추가하여 SpringConfig는 AOP 로직을 적용하지 않도록 설정
    @Around("execution(* hello.hellospring..*(..)) && !target(hello.hellospring.SpringConfig)")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        // ... AOP 로직
    }
}
```