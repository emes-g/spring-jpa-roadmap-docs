## 순수한 단위 테스트
### 개념
> **스프링 컨테이너(서버)를 띄우지 않고, 내가 작성한 비즈니스 로직(특정 클래스나 메서드)만을 독립적으로 검증**하는 테스트이다.

- **스프링 컨테이너(서버)를 띄우지 않고 테스트하는 것이 목적**이지, **스프링 프레임워크에 의존하지 않겠다는 것이 아니다.**
  - 즉, 스프링 관련 import문(`import org.springframework`)과는 무관하다.

### 등장 배경
> **스프링 컨테이너를 띄우는 통합 테스트가 느리기 때문이다.**  

- 스프링 컨테이너를 띄우는 통합 테스트는 실행 속도가 느리고, 여러 객체가 얽혀 있어 에러의 원인을 찾기 어렵다.
- 따라서 **의존관계를 외부에서 가짜로 주입(mocking)하여, 오직 해당 클래스의 로직만 빠르게 검증하는 '순수한 단위 테스트'가 매우 중요**하다.

---

## Mocking
### 개념
> **단위 테스트를 할 때, 실제 동작하는 객체 대신 테스트용 가짜 객체를 주입하는 방식**이다.

- **오직 해당 클래스의 특정 로직만을 검증하기 위해 사용**한다.
- 이와 무관한 DB 연결, 별도의 컨테이너 로직 등은 고려하지 않는다.
- **익명 클래스를 활용하거나, Mockito 프레임워크를 사용하여 Mocking할 수 있다.**

### 종류
- 스프링 컨테이너 없이 순수 자바 코드로 테스트하는 방법은 크게 두 가지이다.
  - 익명 클래스를 활용한 Mocking
  - Mockito 프레임워크를 활용한 Mocking
- 다음 클래스에 대한 테스트 코드를 하나씩 살펴보자.
```java
public class ClientBean {
    private final ObjectProvider<PrototypeBean> provider;

    public ClientBean(ObjectProvider<PrototypeBean> provider) {
        this.provider = provider;
    }

    public int logic() {
        PrototypeBean prototypeBean = provider.getObject();
        prototypeBean.addCount();
        return prototypeBean.getCount();
    }
}
```

---

## 익명 클래스를 활용한 Mocking
### 익명 클래스란?
> **별도의 자바 클래스(파일)를 만들지 않고, 인터페이스의 구현체를 생성하는 자바 문법**이다.

- 인터페이스를 직접 구현하는 방식이므로, **해당 인터페이스 정의된 모든 추상 메서드를 반드시 오버라이딩하여 구현해야 한다.**
- 따라서, **사용하지 않는 메서드까지 모두 강제로 구현해야 한다**는 단점이 발생한다.
    ```java
    // 익명 클래스 방식의 단점: 불필요한 메서드까지 모두 강제 구현해야 함
    ObjectProvider<PrototypeBean> mockProvider = new ObjectProvider<>() {
        @Override
        public PrototypeBean getObject() throws BeansException {
            return new PrototypeBean(); // 필요한 로직
        }
    
        @Override
        public PrototypeBean getObject(Object... args) throws BeansException {
            return null; // 사용하지 않지만 강제로 구현해야 함
        }
    
        @Override
        public PrototypeBean getIfAvailable() throws BeansException {
            return null; // 사용하지 않지만 강제로 구현해야 함
        }
    
        @Override
        public PrototypeBean getIfUnique() throws BeansException {
            return null; // 사용하지 않지만 강제로 구현해야 함
        }
        // ... 더 많은 메서드들 ...
    };
    ```

### 테스트 코드 예시
- `ObjectProvider` 인터페이스를 테스트 코드 내부에서 직접 구현하여 가짜 객체를 만든다.

```java
@Test
void pureJavaTest() {
    // 1. 가짜 ObjectProvider 객체 생성 (Mocking)
    ObjectProvider<PrototypeBean> mockProvider = new ObjectProvider<>() {
        @Override
        public PrototypeBean getObject() {
            return new PrototypeBean(); // 요청이 오면 단순히 새 객체를 반환하도록 조작
        }
        // ... (나머지 메서드는 null 반환 등으로 단순 처리) ...
    };

    // 2. 가짜 객체를 주입하여 ClientBean 생성
    ClientBean clientBean = new ClientBean(mockProvider);

    // 3. 로직 테스트 실행 (스프링 컨테이너 실행 없이 성공)
    int count = clientBean.logic();
    assertThat(count).isEqualTo(1);
}
```

#### **그냥 `new ObjectProvider<>()`만 사용하면 안돼?**

- 애초에 `ObjectProvider`는 구현체(구체적인 클래스)가 아니라 인터페이스다.
- **자바 문법상 인터페이스는 추상적인 설계도일 뿐이므로, `new` 키워드를 사용하여 직접 인스턴스를 생성할 수 없다.**
- 인터페이스의 기능을 사용하려면 반드시 이를 구현(`implements`)하는 구현체가 존재해야 한다.
- **익명 클래스는 어디까지나 인터페이스의 구현체를 생성하는 자바 문법**으로, 우리는 **테스트에서만 사용할 임시 구현체를 익명 클래스를 통해 제공받는 것**이 목적이다.

---

## Mockito 프레임워크를 활용한 Mocking
### Mockito란?
> **익명 클래스의 단점(모든 추상 메서드를 구현해야 한다)을 해결해주는 테스트 프레임워크**다.

### 테스트 코드 예시
```java
import static org.mockito.Mockito.*;

@Test
void mockitoTest() {
    // 1. 가짜 객체 동적 생성 (모든 메서드는 기본적으로 null, 0 등을 반환하는 상태)
    ObjectProvider<PrototypeBean> mockProvider = mock(ObjectProvider.class);

    // 2. Stubbing (동작 정의)
    // "mockProvider의 getObject() 메서드가 호출되면, new PrototypeBean()을 반환하도록 설정"이라는 동작을 정의
    when(mockProvider.getObject()).thenReturn(new PrototypeBean());

    // 3. 테스트 로직 실행
    ClientBean clientBean = new ClientBean(mockProvider);
    clientBean.logic(); // 내부에서 provider.getObject()가 호출되면, stubbing된 PrototypeBean 객체가 반환됨
}
```
- **`mock()` 함수 하나로 가짜 객체를 만들어 빠르고 독립적인 테스트를 수행한 것**을 확인할 수 있다.

#### `mock()` 메서드의 역할
- `mock({인터페이스명}.class)`를 호출하면, **Mockito가 내부적으로 해당 인터페이스를 구현하는 프록시 객체를 런타임에 동적으로 생성하여(리플렉션 기술을 활용하여) 반환**한다. 
- 따라서 일일이 오버라이딩 코드를 작성할 필요가 없어진다.

#### 필요한 메서드만 오버라이딩한다.
- `mock()`으로 생성된 객체의 메서드는 기본적으로 아무 로직도 실행하지 않으며 기본값(`null`, `0`, `false`, 비어있는 컬렉션 등)을 반환할 뿐이다.
- 기본값인 `null`이 반환되면 테스트를 진행할 수 없으므로, **가짜 객체의 특정 메서드를 호출하면 지정한 결과값을 반환하라**고 동작을 미리 정의해줘야 하는데, 이를 **Stubbing**이라고 한다.

#### 스텁? 테스트 스텁과 테스트 드라이버가 있었는데?
> **통합 테스트 단계에서 모듈 간의 의존성을 테스트할 때 등장하는 그 스텁과 동일한 개념**이다.

- **스텁(Stub) - 하위 모듈(구현체) 대체:**
  - 상위 모듈(인터페이스)을 테스트해야 하는데, **상위 모듈이 호출해야 할 하위 모듈(구현체)을 사용할 수 없는 상태일 때 사용**한다.
  - 즉, 상위 모듈의 요청을 받아 단순히 약속된 고정 결과값만을 반환해주는 **하위 모듈 대용 가짜 객체**이다.
  - 이는 하향식(Top-Down) 통합 테스트에서 사용된다.
- **드라이버(Driver)** - 상위 모듈 대체:
  - 하위 모듈(구현체)을 테스트해야 하는데, **하위 모듈을 호출하고 제어해 줄 상위 모듈이 없는 경우**에 사용한다.
  - 즉, 테스트 대상인 하위 모듈에 데이터를 전달하고 결과를 확인하는 **상위 모듈 대용 임시 제어 프로그램**이다.
  - 이는 상향식(Bottom-Up) 통합 테스트에서 사용된다.

#### Mockito 프레임워크는 Stubbing을 수행한다.
- 우리가 작성한 테스트 코드의 구조를 보면 이유가 명확해진다.
  - 테스트 대상(상위 모듈): `ClientBean`
  - 의존 대상(하위 모듈): `ObjectProvider`
- 우리는 `ClientBean`이라는 **상위 모듈의 로직을 테스트 하기 위해, 하위 모듈인 `ObjectProvider`를 가짜 객체로 만들고 특정 메서드를 호출하면 특정 값(`new PrototypeBean()`)을 반환하라**는 동작을 하도록 지정했다.
- 즉, **상위 모듈을 테스트하기 위해 하위 모듈을 대체(Stub)했으므로** 우리는 Stubbing을 수행한 것이다.

### 거대한 인터페이스로 Mocking하면 안 될까?
> **아니, 어차피 Mockito 프레임워크 사용하면 `mock()` 메서드 하나로 가짜 객체를 쉽게 만들 수 있는데, 그러면 굳이 `ObjectProvider` 말고 AC로 Mocking해도 되는거 아닐까?**

- 즉, **Mockito 프레임워크로 `ApplicationContext`를 Mocking해서 `ac.getBean()` 호출 시 `new Prototype()`을 반환하도록 하면 똑같은 거 아닌가**라는 합당한 의문이다.
- 결론부터 말하면 맞는 말이긴 한데, 그렇게 구현해서는 안 된다.

#### 이유: 프로덕션 코드에서의 ISP 위반
- Mockito 프레임워크 사용하면 `mock()` 메서드 하나로 가짜 객체 자체는 쉽게 만들 수 있다.
- 하지만 객체지향적인 관점에서, 고작 **프로토타입 빈 하나를 찾기 위해** 애플리케이션의 설정과 수많은 빈을 관리하는 **거대한 컨테이너 전체를 의존성으로 넘겨주는 것은 ISP를 위반하는 일**이다.
- 어디까지나 **ISP 위반으로 문제가 되는 핵심 지점은 테스트 코드가 아니라 실제 프로덕션 코드**라는 사실을 인지해야 한다.
  - **프로덕션 코드(`ClientBean`)에서 AC를 주입받도록 설계했기 때문에, 테스트 코드에서도 어쩔 수 없이 AC를 Mocking해야 하는 상황이 발생한 것**이다.
  - 즉, 테스트 코드는 프로덕션 코드가 설계된 형태를 그대로 의존해서 작성된다.

#### 왜 프로덕션 코드에서 AC를 주입받는 것이 ISP 위반일까?
- ISP의 핵심은 **클라이언트는 자신이 사용하지 않는 메서드에 의존하지 않아야 한다**는 것이다.
- `ClientBean`은 내부적으로 프로토타입 빈을 하나 조회해서 가져오는 기능 딱 하나만 필요하다.
- 하지만, AC를 통째로 주입받게 되면, 빈 조회 기능 외에도 다음과 같은 수많은 기능들에 대한 접근 권한을 쥐게 된다.
  - 환경 변수 및 프로파일 조회 (`getEnvironment`)
  - 애플리케이션 이벤트 발생 (`publishEvent`)
  - 파일 리소스 읽기 (`getResource`)
- 이는 단순히, ISP를 위반하는 것을 넘어서 **무엇을 Stubbing해야 하는가에 대한 명확성마저 감소시킨다.**  
- 따라서 `ObjectProvider`처럼 **특정 목적(DL)을 수행하는 기능만을 가진 가벼운 인터페이스를 주입받는 것이 훨씬 객체지향적인 설계**이다.