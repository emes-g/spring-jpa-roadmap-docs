## 다양한 의존관계 주입 방법
### 스프링의 라이프사이클
> **스프링 컨테이너는 시작될 때 크게 2단계(빈 생성 → DI)를 거친다. 이 흐름을 알아야 주입 방식을 이해할 수 있다.**

1. **빈 생성 단계:** 객체를 생성한다. (이때 **생성자**가 호출된다.)
2. **DI 단계:** 객체 생성이 끝난 후, `@Autowired`가 붙은 수정자(Setter)나 필드에 의존관계를 주입한다.

### 생성자 주입 (권장)
> **생성자 호출 시점(객체를 생성하는 시점)에 의존관계를 주입하는 방식이다.**

#### 특징
- **딱 1번만 호출되는 것이 보장**되기 때문에 **불변**, **필수** 의존관계에 사용한다.
- 생성자가 딱 1개만 있으면 `@Autowired`를 생략해도 된다. (스프링이 알아서 자동 주입해 준다.)

### 수정자 주입
> **자바빈 프로퍼티 규약(Getter/Setter)을 사용하여 의존관계를 주입하는 방식이다.**

#### 특징
- **빈이 생성된 이후에 자동으로 호출되어 주입**되기 때문에 **선택**, **변경** 가능성이 있는 의존관계에 사용한다.

#### 단점
- 수정자 주입을 하려면 Setter를 `public`으로 열어두어야 한다.
- 때문에 런타임에 의존관계가 변경될 수 있기 때문에 **불변성이 보장되지 않는다**는 단점이 존재한다.

### 필드 주입
> **필드에 의존관계를 주입하는 방식이다.**

#### 단점: 순수 자바 코드로 단위 테스트를 할 수 없다.
```java
public class OrderServiceImpl implements OrderService {
    @Autowired
    private MemberRepository memberRepository; // 필드 주입

    public void createOrder() {
        memberRepository.save();
    }
}

class OrderServiceTest {
    @Test
    void fieldInjectionTest() {
        // 스프링 컨테이너(ac) 없이 순수 자바로 객체 생성
        OrderServiceImpl orderService = new OrderServiceImpl();

        // 필드가 private이라서 외부에서 의존성(구현체)을 넣어줄 방법이 아예 없음
        // orderService.memberRepository = new MemoryMemberRepository(); // 컴파일 에러

        orderService.createOrder(); // NPE 발생
    }
}
```
- **필드 주입을 사용하면 스프링 컨테이너(DI 프레임워크) 없이 테스트가 불가능**한 것을 확인할 수 있다. 
  - 순수한 자바 코드이므로 당연히 `@Autowired`가 동작하지 않기 때문이다.
- 테스트를 하려면 결국 `public` Setter를 만들어야 하는데 그럴 바에는 수정자 주입을 쓰는게 낫고, 만약 불변성도 지키고 싶다면 생성자 주입을 써야 한다.
- 물론, 통합 테스트 환경(`@SpringBootTest`)을 구축해도 되긴 하지만, 이 경우 순수한 자바 코드에 비해 훨씬 무거워진다.

#### 필드 주입이 예외적으로 허용되는 곳
- **테스트 코드:** 남들이 가져다 쓸 일이 없으므로 편하게 써도 된다.
- **설정 파일:** **설정 파일 내부에서만 사용할 특별한 빈을 가져올 때** 간혹 사용한다.
    ```java
    @Configuration
    public class AppConfig {
    
        @Autowired 
        private MemberRepository memberRepository; // 설정 파일 내부에서만 사용
    
        @Bean
        public OrderService orderService() {
            return new OrderServiceImpl(memberRepository);
        }
    }
    ```

### 일반 메서드 주입
> **아무 메서드에 의존관계를 주입하는 방식이지만, 사용할 일이 없다.**

---

## 다양한 의존관계 주입 옵션
> **하지만 주입할 스프링 빈이 없어도 애플리케이션이 동작해야 할 때가 있다.**

- 그런데 `@Autowired`만 사용하면 디폴트가 `required = true`기 때문에 주입 대상이 없는 경우 오류가 발생한다.
- 이때 **의존관계 자동 주입에서 주입 대상이 없을 경우, 선택할 수 있는 옵션이 3가지가 존재**한다.

### 1. @Autowired(required = false)
> **자동 주입할 대상이 없으면 Setter 메서드 자체가 호출되지 않는다.**
```java
@Autowired(required = false)
public void setNoBean1(Member member) {
    System.out.println("setNoBean1 = " + member);
}

// 아무 결과도 출력되지 않음 (애초에 메서드 자체가 호출되지 않았기 때문)
```

### 2. @Nullable
> **자동 주입할 대상이 없으면 null이 주입된다.**
```java
@Autowired
public void setNoBean2(@Nullable Member member) {
    System.out.println("setNoBean2 = " + member);
}

// setNoBean2 = null
```

### 3. Optional<>
> **자동 주입할 대상이 없으면 Optional.empty가 입력된다.**
```java
@Autowired
public void setNoBean3(Optional<Member> member) {
    System.out.println("setNoBean3 = " + member);
}

// setNoBean3 = Optional.empty
```
- `@Nullable`과 `Optional`은 스프링 전반에 걸쳐서 지원된다.

---

## 생성자 주입을 권장하는 이유
> **생성자 호출 시점에 한 번만 호출되므로 객체의 불변성이 보장되고, 컴파일 타임에 의존관계 누락을 파악할 수 있기 때문이다.**

### 객체의 불변성 보장 (Immutability)
- **불변의 필요성:**
  - 대부분의 의존관계는 애플리케이션 시작 후 종료 시점까지 변할 일이 없고, 시스템의 안정성을 위해 변해서도 안 된다.
- **생성자 주입의 특징:**
  - 객체를 생성하는 시점(생성자 호출 시점)에 딱 1번만 호출되는 것이 보장된다.
  - **이후에는 의존관계를 변경할 수 있는 메서드(Setter 등)를 제공하지 않으므로 불변하게 설계할 수 있다.**

### 의존관계 누락 방지 (순수 자바 단위 테스트)
- **순수 자바 테스트의 필요성:**
  - 프레임워크(DI 컨테이너)를 띄우지 않고, **순수 자바 코드만으로 빠르게 단위 테스트를 수행하는 것은 매우 중요**하다.
- **누락 방지 효과:**
  - 순수 자바 테스트 코드 작성 시, 개발자가 직접 구현체를 생성하고 조립해야 한다.
  - 이때 **생성자 주입을 사용하면, 필수 의존관계가 누락되었을 때 자바 컴파일러가 에러를 발생**시켜 준다. (**컴파일 오류는 가장 좋은 오류이다**)

### 생성자 주입은 순수한 자바 언어의 특징을 가장 잘 살리는 DI 방법이다.
> **생성자 주입을 사용하면 프레임워크(DI 컨테이너)의 도움 없이 순수 자바의 `new`, `final` 키워드 그리고 컴파일러의 문법 검사 기능만으로 객체지향적인 코드를 작성할 수 있기 때문이다.**

- 불변과 누락 방지라는 장점 자체가 곧 **자바 언어가 제공하는 문법적 규칙(컴파일러)을 그대로 활용한다**는 의미이다.
- 스프링 프레임워크의 특별한 기능(Reflection, `@Autowired` 등)에 기대지 않고, **자바에서 제공하는 기본 기능만으로도 안전성을 확보한다**는 뜻이다.

#### 컴파일 에러를 통한 누락 방지
- Setter 주입이나 필드 주입은 객체를 생성하는 것(`new`) 자체는 막지 못하기 때문에, 런타임 시점에서야 `NPE`(누락) 오류를 확인할 수 있다.
- 반면, **생성자 주입은 생성자의 파라미터로 의존관계를 주입받아야 하므로, 객체를 생성하는 시점에 반드시 파라미터를 제공받아야 한다.**
- 즉, 생성자에 객체를 넘기지 않으면 컴파일러에 의해 실행(런타임) 자체가 방지된다.

#### `final` 키워드 사용 가능
- 자바에서 **`final` 키워드가 붙은 필드는 반드시 선언 시점이나 생성자 내부에서만 값을 할당할 수 있다.**
- 수정자 주입이나 필드 주입은 생성자 호출 이후에 값을 할당하는 방식으로, `final` 키워드를 사용할 수 없다.
- 오직 생성자 주입만이 `final` 키워드를 사용할 수 있으며, 이를 통해 **초기화가 누락되거나 외부에서 값이 변경되는 것을 자바 문법 레벨에서 완벽하게 차단**한다.

### 결론
> **대부분의 의존관계는 불변이므로 '생성자 주입 + `final` 키워드' 조합의 사용을 권장한다.** 

- 생성자 주입을 디폴트, Setter 주입을 옵션으로 사용하도록 한다. **(필드 주입은 애초에 고려하지 말자.)**

---

## 롬복 (Lombok)
> **Boilerplate code(반복적으로 작성해야 하는 뻔한 코드)를 어노테이션 하나로 자동 생성해주는 라이브러리**

### @RequiredArgsConstructor
> **`final` 필드를 모아서 해당 필드를 파라미터로 받는 생성자를 자동으로 만들어준다.**
```java
@Component
// @RequiredArgsConstructor: final이 붙은 필드를 파라미터로 받는 생성자를 코드로 생성해 준다.
public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    // 즉 MemberRepository 타입과 DiscountPolicy 타입을 파라미터로 받는 생성자가 만들어진다.
    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
    
    // ...

}
```
- 개발자가 다형성을 활용하기 위해 **필드 자체를 구체 클래스가 아닌 인터페이스 타입(`DiscountPolicy`)으로 선언**해두었기 때문에, **롬복이 생성하는 생성자의 파라미터도 자연스럽게 인터페이스와 동일한 타입**이 된다.

### 주의점
> **`Enable annotation processing` 옵션을 켜줘야 동작한다.**

- 롬복은 런타임이 아니라 **컴파일 시점에 소스코드의 어노테이션(`@Getter` 등)을 읽어서 실제 자바 코드를 끼워 넣는 방식(Annotation Processor)으로 동작**한다.
- 따라서 **어노테이션 프로세스 작동을 허용해야 롬복이 코드를 생성할 수 있다.** (Settings에서 설정 가능)

---

## Gradle 의존성 설정
### 의존성 추가 방식
> **라이브러리가 언제 필요한지에 따라 Gradle에서 의존성을 추가하는 방식이 다르다.**

#### 1. `implementation` (both)
- 컴파일(코드 작성) 중에도 해당 라이브러리를 직접 참조하고, 런타임(실행) 중에도 필요한 경우에 사용한다.
- 예) Spring Web, Jackson 등 대부분의 라이브러리

#### 2. `compileOnly`
- **코드를 짜고 빌드할 때까지만 필요**하고, 완성된 프로그램이 구동될 때는 필요없는 경우에 사용한다.
- 예) Lombok: 코드를 생성해주고 나면 런타임에는 필요가 없다.

#### 3. `runtimeOnly`
- 개발자가 코드를 작성할 때는 직접 호출할 일이 없지만, **프로그램 실행 시 백그라운드에서 필요한 경우에 사용**한다.
- 예) DB Driver(MySQL, Oracle 등): JDBC로 코딩하므로 드라이버를 직접 컴파일 시점에 참조하지 않는다.

### 롬복의 동작 원리와 implementation
#### 동작 원리
- 롬복은 런타임에 동작하는 라이브러리가 아니다.
- **컴파일 시점에 개입하여 특정 어노테이션에 맞는 자바 코드를 끼워 넣도록 하는 방식(Annotation Processor)으로 동작**한다.
- 즉, 어노테이션 프로세스 엔진이 작업을 수행한다.

#### implementation의 한계
- 최신 Gradle 정책상, 명시적으로 `annotation processor`를 활성화하지 않으면 해당 엔진이 동작하지 않는다.
- 따라서, `implementation`만 추가하면 해당 엔진이 켜지지 않기 때문에, 의도한 자바 코드 또한 실행되지 않는다.

### 무엇으로 의존성을 추가할 것인가?
> **같은 라이브러리를 추가하더라도 어떤 도구를 사용하느냐에 따라 Gradle 설정 결과가 달라질 수 있으므로 주의해야 한다.**

#### `spring.io` (Spring Initializer)
- 스프링 공식 프로젝트 생성 도구로, 각 라이브러리별 특성과 요구사항을 정확히 파악하고 있다.

#### `add starters` (IDE 편의 기능)
- IDE 편의 기능으로, 개별 라이브러리의 특수한 빌드 요구사항까지 디테일하게 고려하지는 못한다.
- **가장 보편적인 의존성 추가 방식인 `implementation`으로 일괄 처리해버리기 때문에, 핵심 설정 등이 누락되어 라이브러리가 정상 동작하지 않을 수 있다.**

#### 결론
- Lombok, MapStruct, QueryDSL 등 **어노테이션 프로세서 기반이거나 특수한 빌드 설정이 필요한 라이브러리를 도입할 때는 가급적 `spring.io`를 통해 의존성을 추가하는 것이 좋다.**
- 만약 개발 도중 수동으로 추가해야 한다면, `add starters` 대신 `spring.io`에 의존하는 것이 좋다. 

---

## `@Autowired`의 조회 원리와 조회한 빈이 여러 개인 경우
### `@Autowired`의 조회 원리
- `@Autowired`는 기본적으로 **스프링 컨테이너에서 빈의 이름이 아닌 타입(Type)을 기준으로 조회하여 의존관계를 주입**한다.
- 내부적으로 `ac.getBean(Type.class)`와 유사하게 동작한다.

### 조회한 빈이 여러 개일 경우
> **어떤 구현체를 주입해야 할지 판단할 수 없어 `NoUniqueBeanDefinitionException` 예외를 발생시킨다.**

- **문제 상황:**
  - 이름만 다르고 완전히 똑같은 타입의 스프링 빈이 2개 있는 경우
  - 하나의 인터페이스를 구현한 자식 빈 객체가 스프링 컨테이너에 여러 개 등록되어 있는 경우
- **예시:**
  - 객체지향 설계 원칙(DIP, OCP)을 지키기 위해, `OrderServiceImpl` 같은 클라이언트는 구현체(`Rate`, `Fix`)가 아닌 역할(`DiscountPolicy`)에 의존하도록 코드를 작성했다고 하자.
  - 스프링이 `@Autowired`를 통해 해당 타입의 빈을 찾을 때, 해당 역할의 구현체들인 `Rate`와 `Fix` 모두 타입 조회 대상에 포함된다.
  - 빈이 2개 이상 발견되므로, 스프링은 어떤 구현체를 주입해야 할지 판단할 수 없어 `NoUniqueBeanDefinitionException` 예외를 발생시킨다.

---

## `@Autowired`가 조회한 빈이 여러 개인 경우의 해결 방법
### 1. `@Autowired`의 기본 매칭 규칙: 타입 → 필드명
> **타입 매칭을 먼저 시도하고, 그 결과로 여러 빈이 있을 때 필드명 매칭을 시도한다.**

- 타입 매칭: 가장 먼저 주입받을 타입과 일치하는 빈을 찾는다. 
- 이름 매칭: 타입이 같은 빈이 2개 이상 발견되면, 주입받는 곳의 필드명(혹은 파라미터명)과 동일한 이름의 빈을 찾아 주입한다.

### 2. `@Qualifier`: 추가 구분자 제공
> **빈 이름이나 타입 외에 추가적인 단서를 달아주어 명시적으로 매칭하는 방법이다.**

- **빈을 등록할 때, 의존대상에 주입할 때 모두 단서(`@Qualifier("mainDiscountPolicy")`)를 달아주어야 한다.**

#### @RequiredArgsConstructor와 @Qualifier를 같이 쓰고 싶은 경우
```java
@Component
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {
    @Qualifier("mainDiscountPolicy")    // @MainDiscountPolicy와 같은 커스텀 어노테이션은 적용할 수 없음 (직접 생성자로 만들어야 함)
    private final DiscountPolicy discountPolicy; // 롬복이 생성자에 @Qualifier를 알아서 붙여줌
}
```
- 필드 위에 `@Qualifier("단서명")`을 적어두면, 롬복이 생성자를 만들 때 해당 어노테이션을 파라미터에 알아서 달아준다.
- **참고로, `@Qualifier`를 메타 어노테이션으로 갖는 커스텀 어노테이션의 경우, 위처럼 혼용해서 사용할 수 없고 생성자를 직접 만들어야 한다.**

### 3. `@Primary`: 기본값 지정 
> **타입이 같은 빈이 여러 개 있을 때, 최우선으로 주입될 빈(기본값)을 지정하는 어노테이션이다.**

- **한계점:**
  - 무조건 `@Primary`가 붙은 **하나의 빈만 기본으로 주입된다.**
  - 즉, **특정 로직에서는 다른 빈을 주입받고 싶은 경우, `@Primary`만으로 해결할 수는 없다.**
  - 다른 빈을 등록할 때와 주입받고 싶은 곳에 `@Qualifier`를 명시하여 해결해야 한다.

### @Primary와 @Qualifier의 우선순위
- 스프링에서는 항상 **자동보다는 수동이, 범용적인 것보다는 상세한 것이 우선권**을 가진다.
- 따라서 기본값으로 작동하는 `@Primary`보다, 개발자가 직접 이름을 지정한 `@Qualifier`가 더 높은 우선순위를 가져간다.

---

## @Qualifier의 문자열(String) 의존성 문제
> `@Qualifier("~")`와 같이 괄호 안에 문자열을 직접 적어넣으면, **오타가 발생해도 컴파일 타임에 에러를 발견할 수 없다.**

- **이유:** javac는 문자열 내부의 텍스트가 실제로 존재하는 빈 이름인지 검증하지 않기 때문이다.
- **결과:** 런타임에 DI 과정에서 빈을 찾지 못해 애플리케이션이 종료된다.
- **해결책:**
  - **컴파일 타임에 타입 체크가 가능하도록 `@Qualifier`가 포함된 커스텀 어노테이션을 만들어 활용**한다.
  - 예) `@MainDiscountPolicy`라는 어노테이션을 직접 생성

---

## 컬렉션을 이용한 다중 빈 조회와 전략 패턴
### 컬렉션(Map, List)을 이용한 다중 빈 조회
- 의도적으로 특정 타입의 스프링 빈이 모두 필요한 상황이 있다.
- 스프링에서 의존관계를 주입받을 때 **파라미터를 `Map`이나 `List`와 같은 컬렉션으로 지정하면, 컨테이너에 등록된 해당 타입의 빈을 모두 찾아서 한 번에 주입**해 준다.
  - `Map<String, DiscountPolicy>`: 스프링 빈 저장소처럼 스프링 빈 이름을 키로, 실제 인스턴스를 값으로 매핑하여 주입한다.
  - `List<DiscountPolicy>`: 해당 타입의 모든 빈 객체를 리스트 형태로 모아서 주입한다.
- 이는 **클라이언트의 요청이나 런타임 상황에 따라 동적으로 사용할 빈을 선택해야 할 때** 매우 편리하다.
- 분기 처리를 통해 빈을 고를 필요 없이, `Map`으로 모든 빈을 주입받아 두고, 넘겨받은 식별자(빈 이름)를 키로 사용하여 맵에서 즉시 해당 객체를 꺼내 실행하면 된다.

### 전략 패턴
> **런타임에 알고리즘이나 동작 방식(전략)을 동적으로 선택하고 교체할 수 있게 해주는 디자인 패턴**이다.

- 예) 클라이언트가 '고정 금액 할인'을 받을지, '비율 할인'을 받을지 런타임에 동적으로 처리해야 하는 경우에 사용하면 좋다.

---

## 순수 자바 팩토리 패턴 vs 스프링 Map 주입
### 순수 자바 팩토리 패턴의 5가지 구성 요소
> **문자열 요청을 받아 동적으로 할인 정책을 적용하는 상황을 순수 자바로 구현하려면 다음의 5가지 요소가 필요**하다.

1. **전략 인터페이스:** 할인 정책(`DiscountPolicy`)과 같은 공통 규격
```java
public interface DiscountPolicy {
    int discount(int price);
}
```

2. **구체적인 전략들:** 고정 할인(`Fix`), 비율 할인(`Rate`) 등 실제 구현체
```java
public class FixDiscountPolicy implements DiscountPolicy {
    @Override
    public int discount(int price) { return 1000; }
}

public class RateDiscountPolicy implements DiscountPolicy {
    @Override
    public int discount(int price) { return price * 10 / 100; }
}
```

3. **전략을 결정해주는 팩토리:** 요청 문자열을 받아 분기를 통해 어떤 구현체를 꺼내줄지 조립하고 결정하는 구성 영역
```java
public class DiscountPolicyFactory {
    public static DiscountPolicy getDiscountPolicy(String discountCode) {
        if ("FIX".equals(discountCode)) {
            return new FixDiscountPolicy(); 
        } else if ("RATE".equals(discountCode)) {
            return new RateDiscountPolicy();
        } else {
            throw new IllegalArgumentException("알 수 없는 할인 코드");
        }
    }
}
```

4. **전략을 사용하는 곳:** 비즈니스 로직을 수행하는 서비스
```java
public class OrderService {
    
    public int calculatePrice(int price, DiscountPolicy discountPolicy) {
        return price - discountPolicy.discount(price);
    }
}
```

5. **전략을 선택하는 클라이언트:** `FIX`나 `RATE` 등 자신이 원하는 특정 전략을 쓰겠다고 문자열 데이터로 팩토리에 요청하는 주체
```java
// 5. 클라이언트 요청
public class Main {
    public static void main(String[] args) {
        OrderService orderService = new OrderService();
        
        // 클라이언트가 "FIX" 할인을 요청함
        String request = "FIX";
        DiscountPolicy policy = DiscountPolicyFactory.getDiscountPolicy(request); // 팩토리에서 전략 꺼냄
        int finalPrice = orderService.calculatePrice(20000, policy); // 서비스에 넘겨서 실행
    }
}
```

### 순수 자바 팩토리 패턴의 한계
- **새로운 기능(`VipDiscountPolicy`)이 추가될 때마다 개발자가 직접 팩토리 클래스를 수정해야 하는 번거로움이 존재**한다.

### 스프링 Map 주입 방식
> **스프링 컨테이너의 빈 자동 주입(`@Autowired` to Map) 기능을 사용하면 기존 구조에 있던 팩토리 클래스 자체를 아예 작성할 필요가 없다.**
```java
@Component
@RequiredArgsConstructor
public class OrderService {
    // 스프링이 DiscountPolicy 타입의 모든 빈을 알아서 Map에 담아 주입해 줌
    // 예: {"fixDiscountPolicy": FixDiscountPolicy객체, "rateDiscountPolicy": RateDiscountPolicy객체}
    private final Map<String, DiscountPolicy> policyMap; 

    // 클라이언트가 "fixDiscountPolicy"라는 문자열(key)을 넘김
    public int calculatePrice(int price, String discountCode) {
        // if-else 팩토리 로직 없이, Map에서 즉시 꺼내서 실행
        DiscountPolicy policy = policyMap.get(discountCode);
        return price - policy.discount(price);
    }
}
```
- **개선점:** 새로운 전략 클래스(`VipDiscountPolicy`)가 추가되더라도 `@Component`만 붙이면 스프링 컨테이너가 알아서 `Map`에 객체를 주입해준다.
- **의의:**  
    - 순수 자바 코드로 전략 패턴을 구현하려면 각 전략(구현체)을 생성하고 관리하는 조립용 코드나 팩토리 클래스가 추가로 필요하다.
    - 하지만 스프링은 **컨테이너가 생성한 빈을 `Map`으로 한 번에 묶어 주입해 주는 기능이 있으므로, 별도의 복잡한 구현 없이 전략 패턴을 달성할 수 있다.**
    - 덕분에, 순수 자바에서는 필연적으로 발생하던 **구성 영역(팩토리) 코드를 수동으로 변경해야 하는 작업이 스프링에서는 완벽하게 자동화**되었다.

---

## 스프링 컨테이너 생성자
> **스프링 컨테이너는 생성자에 클래스 정보를 받는데, 인자로 넘긴 클래스들이 스프링 빈으로 등록된다.**

```java
// 다음 코드의 동작 과정을 살펴보자
new AnnotationConfigApplicationContext(AutoConfig.class, DiscountService.class);
```
1. `new AnnotationConfigApplicationContext()`를 통해 스프링 컨테이너가 생성된다.
2. 인자로 넘긴 `AutoAppConfig`와 `DiscountService` 클래스가 우선적으로 스프링 빈으로 등록된다.
3. 스프링 내부에서 설정 정보를 분석하는 과정이 실행되는데, 이때 앞서 등록된 `AutoAppConfig`에 `@ComponentScan`이 붙어 있는 것이 확인된다.
4. `@ComponentScan`에 따라 `@Component`가 붙은 클래스들을 잇달아 스프링 빈 설정 정보에 (`BeanDefinition`의 형태로) 추가로 등록한다.
5. 모든 클래스들이 `BeanDefinition`으로 등록이 되었다면, 이 정보를 바탕으로 하나씩 빈 객체를 생성한다.
6. 이때 객체들의 생성 순서는 의존관계에 따라 동적으로 결정된다.

> **즉, 의존관계에 상관없이 `BeanDefinition`은 모두 일괄 등록되며, 이후 실제 빈 객체를 생성할 때 필요로 하는 의존성을 먼저 파악하여, 연쇄적으로 객체를 생성하고 주입하는 방식으로 동작한다.**

---

## 빈 등록 방법을 어떻게 결정해야 할까?
### 자동 빈 등록을 기본으로 사용하는 이유
- **수동 등록은 관리할 빈이 많아질수록 부담이 된다.**
- 자동 등록(`@Component`, `@Autowired`)을 사용해도 OCP, DIP와 같은 객체지향 설계 원칙을 지킬 수 있다.
- 일반적인 업무 로직은 **숫자가 많고 패턴이 명확하기 때문에 이러한 자동 빈 등록이 권장**된다.

### 업무 로직과 기술 지원 로직
- **업무 로직:** 
  - 웹 요청 처리, 핵심 비즈니스 연산, 데이터 저장 등 **사용자의 요구사항을 처리**하는 기능이다.
  - 예) Controller, Service, Repository 등
- **기술 지원 로직:** 
  - **업무(비즈니스) 로직이 원활하게 동작하도록 뒤에서 돕는 공통 인프라 기술**이다.
  - 예) DB 연결, 공통 로그 처리, 보안 등 

### 수동 빈 등록은 언제 필요할까?
> 업무 로직과 달리 **설정 파일에 수동으로 등록하여 구조를 명확하게 드러내는 것이 유리할 때**이다.

#### 직접 개발한 기술 지원 로직
- 기술 지원 로직은 숫자는 적지만 어플리케이션 전반에 광범위하게 영향을 미치고,
- **문제가 생겼을 때 원인을 찾거나 제대로 적용되고 있는지를 파악하기 어렵기 때문**이다.

#### 다형성을 적극 활용하는 비즈니스 로직 
- 앞서 배운 `Map<String, DiscountPolicy>`처럼 전략 패턴을 활용할 때, **자동 주입을 사용하면 어떤 구체적인 빈들이 컨테이너에 등록되어 맵에 들어오는지 한눈에 파악하기 어렵다.**
- 이를 해결하기 위해서는 특정 인터페이스들의 구현체를 한 설정 파일(`DiscountPolicyConfig`)에 등록하는 것이 좋다.
- 혹은 만약 자동으로 하더라도, 특정 패키지에 깔끔하게 모아두어 **한 눈에 파악할 수 있도록** 만들어 둬야 한다.

### 자체적으로 제공하는 기술 지원 로직은 수동 등록하기 보다 의도대로 사용하는 것이 중요하다.
- 스프링이나 스프링 부트가 **자체적으로 제공하는 수많은 기술 지원 빈(`DataSource`, `TransactionManager` 등)은 `application.properties` (또는 `.yml`) 설정만으로 자동 등록되도록 설계되어 있다.**
- 이런 부분은 굳이 수동으로 재등록할 필요 없이, **스프링의 의도대로 자동 기능을 편리하게 사용하는 것**이 정답이다.

### 요약
1. 편리한 자동 등록(`@Component`, `@Autowired`)을 기본으로 사용한다.
2. 기술 지원 객체는 **애플리케이션 전반에 영향을 미치기 때문에 수동 등록하여 관리하는 것**이 좋다. (단, 직접 개발한 기술 지원 로직에 한함)
3. 다형성을 적극 활용하여 여러 구현체를 주입받는 비즈니스 로직은 **한눈에 파악하기 쉽도록 수동 등록을 고민**해 보면 좋다.



---

## 인텔리제이 단축키
- `ctrl` + `f12` (File Structure)
  - 해당 클래스가 가진 필드, 메서드, 생성자 등을 한눈에 파악하고 빠르게 검색하여 이동할 수 있다.
  - 이때 롬복이 자동으로 생성해 준 메서드(Getter, Setter 등)가 잘 만들어졌는지 확인할 때도 매우 유용하다.
- `ctrl` + `b` (선언부 이동)
  - 해당 메서드나 클래스가 **선언된 곳(주로 인터페이스나 추상 클래스)으로 이동**한다.
  - 해당 기능의 명세를 확인할 때 사용한다.
- `ctrl` + `alt` + `b` (구현체 이동)
  - 해당 인터페이스의 구현체로 이동한다.
  - 다형성을 활용하여 구현체가 여러 개일 때, 실제 로직이 작성된 곳을 찾기 위해 사용한다.