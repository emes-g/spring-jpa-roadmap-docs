## 스프링 빈과 의존관계 (Component Scan)

### 1. @Autowired와 의존성 주입 (DI)
- **정의:** 생성자에 `@Autowired`를 붙이면, 스프링 컨테이너가 관리하는 스프링 빈(Bean)을 찾아 자동으로 연결(주입)해줌
- **표현:** "스프링 컨테이너에 등록된 빈 객체를 자동으로 주입해준다"는 표현이 정확함
- **비유:** 객체 간의 관계선(연결고리)을 스프링이 알아서 이어주는 것

### 2. 컴포넌트 스캔 (Component Scan)
- **원리:** `@Component` 애노테이션이 붙은 클래스(포함: `@Controller`, `@Service`, `@Repository`)를 스캔하여 스프링 빈으로 자동 등록하는 과정
- **적용 범위 (Scope):**
    - 원칙적으로 **실행 애플리케이션 클래스(`@SpringBootApplication`이 붙은 파일)가 위치한 패키지와 그 하위 패키지**만 스캔 대상이 됨
    - 해당 범위를 벗어난 패키지에 클래스를 만들면 스프링 빈으로 등록되지 않음

--- 

## 스프링 빈 설정과 의존관계 주입(DI) 심화

### 1. IntelliJ 단축키
- **Ctrl + P (Parameter Info):** 메서드 호출 괄호 `()` 안에서 누르면, 어떤 파라미터(타입, 변수명)를 넘겨야 하는지 정보를 보여줌

### 2. 스프링 빈 등록 방식 비교
**자동 등록 (Component Scan)**
- **방법:** 클래스에 `@Component` (또는 `@Controller`, `@Service`, `@Repository`)를 붙임
- **특징:** 스프링이 켜질 때 해당 패키지를 뒤져서 알아서 빈으로 등록함
- **참고:** `Controller`는 웹과 직접 연관되어 있어 주로 자동 등록 방식을 사용함

**수동 등록 (Java Code Configuration)**
- **방법:** `@Configuration`이 붙은 설정 클래스를 만들고, 내부에 `@Bean`이 붙은 메서드로 객체를 반환
- **특징:**
  - 개발자가 직접 조립 코드를 작성함
  - **구현 클래스를 변경해야 할 때(예: MemoryRepo -> DbRepo)** 설정 파일만 수정하면 되므로 매우 편리함

### 3. 의존관계 주입 (DI) 3가지 방법
**1) 생성자 주입 (권장)**
- **특징:** 객체 생성 시점에 생성자를 통해 딱 1번만 호출됨을 보장함
- **장점:**
    - **불변:** 한번 조립되면 바꿀 수 없음 (안전함)
    - **누락 방지:** 필수 의존관계가 없으면 컴파일 에러 발생
    - **테스트 용이:** 순수한 자바 코드로 단위 테스트를 할 때, 개발자가 직접 `new MockRepository()` 등을 넣어줄 수 있음

**2) 필드 주입 (비권장)**
- **형태:** `@Autowired private MemberRepository repository;`
  - **단점 (바꿀 수 없다는 의미):**
      - 스프링 없이 순수 자바 코드로 테스트를 할 때, `private` 필드에 객체를 넣어줄 방법이 없음 (setter나 생성자가 없으므로)
        ```java
        // [필드 주입 방식의 Service]
        public class MemberService {
            @Autowired
            private MemberRepository repository; // private 접근 제어자 때문에 외부에서 접근 불가
        
            public void join(Member member) {
                repository.save(member); // 테스트 할 때, repository가 초기화되지 못해 NPE 발생
            }
        }
        ```
        ```java
        // [테스트 코드]
        @Test
        void join() {
        MemberService service = new MemberService();
        // service.repository = new MemoryRepository(); // 불가능
        
            service.join(member); // NPE 발생
        }
        ```
      - 결국 테스트를 하려면 스프링 컨테이너를 띄워야 하므로 테스트가 무거워짐
- **시점:** 객체 생성(new) 후, 스프링이 리플렉션 기술로 억지로 찔러 넣어줌

**3) Setter 주입**
- **형태:** `setRepository(...)` 메서드를 열어둠
- **단점:** 누군가 실수로 실행 중에 호출하면 의존관계가 바뀔 위험이 있음 (불변성 깨짐)

### 4. 구현체 교체 시나리오 (수동 등록의 장점)
**상황:** `MemoryRepository`를 쓰다가 `DbRepository`로 교체해야 함

- **자동 등록 사용 시:**
    - `MemoryRepository` 코드를 열어 `@Repository`를 지움
    - `DbRepository` 코드를 열어 `@Repository`를 붙임
    - **단점:** 실제 동작하는 애플리케이션 코드를 여러 군데 손대야 함

- **수동 등록 사용 시:**
    - `SpringConfig` 파일만 열어서 `return new Memory...`를 `return new Db...`로 고치면 끝
    - 실제 애플리케이션 코드는 전혀 손대지 않고, **설정 파일 하나만 수정**하면 됨
      - 즉, **설정(Configuration)과 동작(Application) 코드가 명확하게 분리됨**
     

### 5. @Autowired의 동작 원리
- `@Autowired`는 **스프링 컨테이너에 등록된 빈**끼리만 동작함
- 내가 직접 `new`해서 만든 객체(스프링 관리를 받지 않는 객체) 안에서는 `@Autowired`를 써도 작동하지 않음