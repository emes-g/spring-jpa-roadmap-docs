## JPA와 H2 데이터베이스 설정

### 1. JPA와 Spring Data JPA
- **JPA (Java Persistence API):**
    - 자바 객체와 DB 테이블을 매핑해주는 기술 표준 (ORM)
    - 개발자가 SQL을 직접 짜지 않고, **객체를 저장하면 JPA가 알아서 SQL을 만들어 DB에 날려줌**
- **Spring Data JPA:**
    - JPA를 껍데기로 한 번 더 감싸서 편하게 만든 프레임워크
    - `repository` 인터페이스만 만들면 기본적인 CRUD(저장, 조회 등) 로직을 **구현체 없이 자동으로 만들어줌**

### 2. H2 데이터베이스 특징
- **용도:** 교육용, 테스트용, 소규모 개발용으로 적합
- **장점:**
    - 용량이 매우 작고 가벼움
    - 웹 브라우저에서 관리할 수 있는 **Admin 화면(Console)** 제공
    - 별도 설치 없이 라이브러리만으로 구동 가능

### 3. H2 접속 방법 (JDBC URL) 분석
URL 형식에 따라 데이터 저장 위치와 접속 방식이 완전히 달라집니다.

**1) 인메모리 모드 (`jdbc:h2:mem:skala-stock`)**
- **mem:** Memory(램)
- **특징:** 데이터가 RAM(메모리)에만 저장됨
- **휘발성:** 애플리케이션(서버)을 끄면 **데이터가 모두 사라짐**
- **용도:** 아주 가벼운 테스트나 연습용

**2) 임베디드 모드 (파일 직접 접근) (`jdbc:h2:~/test`)**
- **~:** 사용자 홈 디렉토리 (C:\Users\사용자명)
- **특징:** 홈 디렉토리에 `test.mv.db`라는 **파일을 생성**하여 저장함
- **비휘발성:** 서버를 꺼도 파일이 남아있으므로 **데이터가 유지됨**
- **단점:** 애플리케이션이 파일을 직접 잡고(Lock) 있기 때문에, 웹 콘솔 등 다른 곳에서 동시에 접근하려고 하면 **충돌 오류**가 발생할 수 있음

**3) 서버 모드 (`jdbc:h2:tcp://localhost/~/test`)**
- **tcp:** 네트워크 통신 방식
- **특징:** 파일을 직접 여는 게 아니라, 떠 있는 **H2 서버에 접속 요청**을 보내는 방식
- **이유:**
    - 파일 모드로 생성된 DB에 **여러 클라이언트(앱, 웹 콘솔 등)가 동시에 접속**하기 위함
    - 파일 락(Lock) 문제를 피하기 위해 **파일 접근을 H2 서버에게 위임**하고 통신으로 데이터를 주고받음
    - **결론:** 파일에 저장되므로 **비휘발성**이면서, **동시 접속 문제도 해결**한 방식

---
## 순수 JDBC와 데이터 접근의 원리

### 1. 학습 목적 (History)
- **이유:** 실무에서 이렇게 코드를 짜지는 않음(너무 길고 반복적이라).
- **목표:** 
  - 데이터 저장 기술의 역사(**어떤 과정을 거쳐 발전해왔는지**)를 이해하고, 스프링이 감춰준 내부 원리를 파악하기 위함 
  - 편하게 듣고 넘어가면 됨

### 2. 환경 설정 (Configuration)
- **build.gradle 설정**
    - `h2`: 데이터베이스 드라이버
    - `spring-boot-starter-jdbc`: **(중요)** 강의에서는 이것을 사용함.
    - *참고:* `session-jdbc`는 세션 정보를 DB에 저장할 때 쓰는 것이라 용도가 다름. 일반적인 JDBC 학습용으로는 `starter-jdbc`가 맞음.
- **application.properties**
    - `spring.datasource.url`: 접속할 DB 주소
    - `spring.datasource.driver-class-name`: 사용할 드라이버 클래스 (**DB에 맞는 드라이버를 사용하는 것**이 중요!)
    - `username`, `password`: 접속 계정 정보

### 3. 구조적 특징 (OCP 원칙)
- **인터페이스 (`MemberRepository`):** "회원을 저장한다"는 역할은 그대로 유지됨
- **구현체 교체:**
    - 기존: `MemoryMemberRepository` (메모리에 저장)
    - 변경: `JdbcMemberRepository` (DB에 SQL을 날려서 저장)
- **결과:** 스프링 설정(`Config`)만 바꾸면 비즈니스 로직 수정 없이 저장소가 변경됨 (다형성 활용)

### 4. JDBC 핵심 객체 3가지
1.  **Connection (`conn`):**
    - **역할:** DB와 애플리케이션 간의 **연결(Session)**.
    - **비유:** 전화를 걸어서 상대방(DB)과 통화가 연결된 상태.
2.  **PreparedStatement (`pstmt`):**
    - **역할:** SQL문을 담아서 DB에 보내는 **보따리(운송수단)**.
    - **특징:** `?`를 통해 파라미터를 바인딩하여(`setString` 등) SQL Injection 공격을 방지함.
3.  **ResultSet (`rs`):**
    - **역할:** `SELECT` 쿼리 실행 후, DB가 돌려준 **결과 데이터 집합**.
    - **사용:** `rs.next()`로 한 줄씩 커서를 이동하며 데이터를 읽음.

### 5. 주요 로직 흐름
1.  **DataSource 주입:** 스프링한테서 DB 접속 정보를 가진 `dataSource`를 받음.
2.  **커넥션 획득 (`getConnection`):** DB와 소켓 연결을 맺음.
3.  **SQL 작성 및 전송:** `conn.prepareStatement(sql)` -> `setString(파라미터)` -> `executeUpdate()` (실행).
4.  **ID 자동 생성 (`RETURN_GENERATED_KEYS`):**
    - `MemoryRepo`에서는 `sequence++`로 직접 ID를 올렸지만,
    - DB에서는 `AUTO_INCREMENT` 등으로 DB가 알아서 ID를 생성함.
    - `RETURN_GENERATED_KEYS` 옵션을 쓰면, **"방금 DB가 만든 ID가 뭐니?"** 하고 가져올 수 있음.
5.  **자원 해제 (`release`):**
    - `conn`, `pstmt`, `rs`는 외부 리소스(TCP 연결)를 쓰기 때문에 **반드시 닫아줘야 함(`close`)**. 안 그러면 DB 커넥션이 꽉 차서 서버가 죽음.

### 6. 심화 개념 (Tip)
- **CQRS (Command and Query Responsibility Segregation):**
    - **Command (쓰기):** 상태를 변경함 (Create, Update, Delete) -> 보통 리턴값이 없거나 ID만 반환.
    - **Query (읽기):** 상태를 변경하지 않고 조회만 함 (Read) -> 데이터를 반환.
    - **이유:** 읽기 요청이 쓰기보다 훨씬 많으므로, 나중에 시스템을 확장할 때 읽기 전용 DB와 쓰기 전용 DB를 나누는 등 최적화를 위해 개념을 분리함.
- **DataSourceUtils:**
    - 커넥션을 직접 `open/close` 하지 않고, 스프링이 제공하는 `DataSourceUtils`를 통해야 **트랜잭션 동기화**가 안전하게 유지됨.

---

## 데이터베이스 드라이버 (Database Driver)

### 1. 정의
- **역할:** 자바 애플리케이션(Java)과 데이터베이스(DB) 사이의 **통역사(Translator)**
- **필요성:**
    - 자바는 표준 언어(JDBC 인터페이스)로 명령을 내리지만, 각 DB(H2, MySQL, Oracle)는 서로 다른 고유의 통신 방식(프로토콜)을 사용함
    - 따라서 자바의 명령을 **해당 DB가 알아들을 수 있는 언어로 변환**해 주는 라이브러리가 반드시 필요함

### 2. 작동 원리
- **구조:** `Java App` -> **`JDBC Interface`** -> **`DB Driver (구현체)`** -> `Database`
- **예시:**
    - H2 DB를 쓴다면? -> **H2 Driver** (`h2.jar`)가 필요
    - MySQL DB를 쓴다면? -> **MySQL Driver** (`mysql-connector.jar`)가 필요
- **특징:** 개발자는 DB가 바뀌어도 자바 코드를 수정할 필요 없이, **드라이버 설정(build.gradle)만 교체**하면 됨

--- 

## PreparedStatement 파라미터 바인딩 규칙

### 1. 인덱스 시작 번호 (1-based Index)
- **규칙:** 파라미터 인덱스는 **1부터 시작**함
- **주의:** 자바의 배열이나 리스트가 0부터 시작하는 것과 다름 (혼동 주의)
- **예시:** `pstmt.setString(1, ...)` -> 첫 번째 물음표(`?`)를 의미함

### 2. 매핑 순서
- **규칙:** SQL 문장에 적힌 **물음표(`?`)의 순서대로** 매핑됨
- **코드 예시:**
    ```java
    // SQL: 이름(? 1번), 나이(? 2번)
    String sql = "insert into member(name, age) values(?, ?)";
    
    pstmt = conn.prepareStatement(sql);
    
    // 첫 번째 ? (name)
    pstmt.setString(1, "spring"); 
    
    // 두 번째 ? (age)
    pstmt.setInt(2, 20); 
    ```

---
## 객체지향 설계와 스프링의 DI (SOLID 적용 사례)

### 1. 다형성(Polymorphism)의 활용
- **구조:** `MemberService`는 구체적인 클래스가 아닌, **인터페이스(`MemberRepository`)** 에 의존함
- **효과:** 인터페이스를 구현한 어떤 객체(`Memory`, `Jdbc`)든지 바꿔 낄 수 있는 유연한 구조가 됨

### 2. 영역의 분리 (App vs Assembly)
애플리케이션을 **'사용 영역'** 과 **'구성 영역'** 으로 나누는 것이 핵심이다.

- **애플리케이션 로직 (사용 영역):**
    - `MemberService`, `OrderService` 등 실제 비즈니스를 수행하는 코드
    - **특징:** 어떤 리포지토리가 들어오는지 전혀 모름 (변경 없음)
- **어셈블리 / 설정 코드 (구성/조립 영역):**
    - `AppConfig`, `SpringConfig` 등
    - **역할:** 애플리케이션의 "공연 기획자". 배우(구현체)를 섭외하고 배역을 정해주는 역할
    - **특징:** 구현체를 교체할 때 **이 부분만 수정하면 됨**

### 3. OCP (Open-Closed Principle, 개방-폐쇄 원칙)
- **확장에는 열려 있다 (Open):**
    - `Memory`에서 `Jdbc`, `JPA` 등으로 기능을 얼마든지 확장할 수 있음 (구현체 추가)
- **변경에는 닫혀 있다 (Closed):**
    - 기능을 확장해도 **클라이언트 코드(`MemberService`)는 전혀 수정할 필요가 없음**
    - *참고:* 조립하는 코드(Config)는 당연히 수정해야 하지만, 애플리케이션의 핵심 로직은 변경되지 않는다는 의미

### 4. DIP (Dependency Inversion Principle, 의존관계 역전 원칙)
- **정의:** 구체화에 의존하지 말고 **추상화에 의존**해야 한다.
- **적용:**
    - `MemberService` -> `MemberRepository` (인터페이스/추상화) **[O]**
    - `MemberService` -> `JdbcMemberRepository` (구현체/구체화) **[X]**
- **결과:** 스프링 컨테이너(DI)가 외부에서 의존관계를 주입해 줌으로써 이 원칙을 완벽하게 지킬 수 있음
---

## 스프링 통합 테스트와 트랜잭션 (Integration Test)

### 1. 통합 테스트 설정
- **@SpringBootTest:**
    - 스프링 컨테이너를 실제로 띄워서 테스트를 실행함
    - 스프링 빈(Bean)을 모두 스캔하고 등록하므로, 실제 운영 환경과 거의 유사한 상태에서 테스트 가능
    - **단점:** 컨테이너를 띄우는 데 시간이 오래 걸림 (무거움)

### 2. 테스트 코드의 의존성 주입 (Field Injection)
- **방식:** 생성자 주입 대신 `@Autowired`를 필드에 바로 붙여서 사용 (`@Autowired MemberService service;`)
- **이유:**
    - 테스트 코드는 **"가장 끝단"** 에 있는 코드임 (다른 곳에서 이 테스트 클래스를 가져다 쓸 일이 없음)
    - 따라서 생성자를 만들고 주입받는 번거로움 없이, 가장 편한 방식인 필드 주입을 사용해도 무방함

### 3. @Transactional과 DB 롤백
- **배경:** DB는 **트랜잭션(Transaction)** 개념이 있어서, 데이터를 넣어도(`INSERT`) **커밋(Commit)** 하지 않으면 실제로는 반영되지 않음(물론 오토커밋 설정에 따라 바로 반영될 수는 있음)

- **동작 원리 (테스트 환경):**
    1. 테스트 시작 전: 트랜잭션 **시작(Start)**
    2. 테스트 실행: `INSERT` 쿼리 날림 (DB에 임시 저장 상태)
    3. 테스트 종료 후: **강제로 롤백(Rollback)함**
- **장점:**
    - DB에 데이터가 남지 않으므로, 다음 테스트에 영향을 주지 않음
    - `@AfterEach`로 `delete` 쿼리를 날리거나 `clearStore`를 할 필요가 사라짐 (재현성 보장)

---

### ❓ 의문 해결 1: 커밋을 안 하는 것 vs 롤백을 하는 것?
> *"커밋을 하지 않는다 == 롤백을 한다?"*

**엄밀히 말하면 다르지만 결과는 같다.**

1.  **커밋을 안 함:** DB 연결을 끊을 때 커밋되지 않은 데이터는 DB가 알아서 버림 (암시적 롤백)
2.  **롤백을 함:** "방금 한 거 취소해!"라고 명시적으로 명령을 내림

**@Transactional의 동작:**
- 스프링은 테스트가 끝나면 **명시적으로 `ROLLBACK` 명령**을 DB에 날린다.
- "커밋을 안 하고 기다리는 것"이 아니라, "넣었다가 다시 원상복구(취소) 시키는 것"이 정확한 표현이다.

---

### 4. 단위 테스트 vs 통합 테스트
- **단위 테스트 (Unit Test):**
    - **정의:** 스프링 컨테이너나 DB 연결 없이, 순수 자바 코드로 **최소한의 단위(메서드, 클래스)** 만 테스트함
    - **장점:** 실행 속도가 압도적으로 빠름 (밀리초 단위)
    - **철학:** *"좋은 테스트는 단위 테스트일 확률이 높다."* (컨테이너 없이도 로직 검증이 가능해야 좋은 설계임)

- **통합 테스트 (Integration Test):**
    - **정의:** 스프링 컨테이너, DB, 외부 라이브러리 등 실제 환경을 연동해서 테스트함
    - **용도:** 객체 간의 협력이나 DB 트랜잭션 동작 등을 검증할 때 필요

---

### ❓ 의문 해결 2: 스프링을 띄우면 단위 테스트가 아닌가?
> *"스프링 컨테이너를 띄우면 단위 테스트가 아닌건가? 특정 기능을 테스트하려면 의존성을 주입받아야 하는데?"*

**그렇다, 스프링 컨테이너를 띄우는 순간 그것은 '통합 테스트'로 분류된다.**

1.  **순수 단위 테스트:**
    - `MemberService`를 테스트할 때, `MemberRepository`를 스프링에게 달라고 하지 않고 **가짜 객체(Mock)** 를 만들어서 직접 넣어준다.
    - 예: `new MemberService(new FakeMemoryRepository());`
    - 이렇게 하면 스프링 없이도 로직 검증이 가능하다.

2.  **딜레마:**
    - "스프링 빈을 주입받아야만 테스트가 가능한데?" 
      <br>→ 이 상황 자체가 **"내 코드가 스프링 프레임워크에 너무 강하게 결합되어 있다"** 는 신호일 수 있다.
    - 그래서 스프링 없이도 테스트 가능한 구조(POJO)가 지향된다.

---

## JdbcTemplate과 데이터 접근 기술 비교

### 사용 이유 (Why?)
* **순수 JDBC의 문제:** DB 연결(`Connection`), 예외 처리(`try-catch`), 자원 해제(`close`) 등 반복적인 코드가 비즈니스 로직보다 더 길어지는 주객전도 현상 발생.
* **JdbcTemplate의 해결:** "템플릿 콜백 패턴"을 사용하여 반복 코드는 라이브러리가 처리하고, 개발자는 **SQL 작성**과 **파라미터 정의**에만 집중할 수 있게 함.

### 기술 스택 비교 (MyBatis vs JPA vs JdbcTemplate)
실무에서 JdbcTemplate을 여전히 사용하는 이유를 이해하기 위한 비교이다.

| 기술 | 분류 | 특징 및 장단점 | 실무 활용 포인트 |
| :--- | :--- | :--- | :--- |
| **JdbcTemplate** | SQL Mapper (Light) | • 설정이 매우 간편함 (의존성만 추가하면 끝).<br>• 동적 쿼리(조건에 따라 바뀌는 쿼리) 작성은 조금 불편함. | • JPA를 메인으로 쓰더라도, **복잡한 통계 쿼리**나 **가벼운 SQL**을 실행할 때 보조적으로 많이 사용. |
| **MyBatis** | SQL Mapper (Heavy) | • SQL을 자바 코드가 아닌 **XML 파일**로 분리해서 관리.<br>• 복잡한 동적 쿼리 관리에 최적화됨. | • 국내 SI, 금융권, 레거시 시스템에서 주력으로 사용.<br>• 쿼리가 매우 길고 복잡할 때 유리. |
| **JPA** | ORM (Object Mapping) | • SQL을 아예 안 씀 (객체를 바로 DB에 저장).<br>• 생산성은 높으나 학습 곡선이 높음. | • 모던 웹 개발의 주력 기술.<br>• 단순 CRUD(등록/조회/수정/삭제)에 압도적으로 유리. |

---

## 의존성 주입 (DI) 참고 사항

### 생성자 주입의 생략
스프링 빈으로 등록된 클래스에서 **생성자가 딱 하나만 존재할 경우**, `@Autowired` 어노테이션을 생략해도 스프링이 자동으로 의존성을 주입해 준다.

```java
// [과거/정석]
@Autowired // 생략 가능
public MemberRepository(DataSource dataSource) {
    this.template = new JdbcTemplate(dataSource);
}

// [최근 트렌드]
// 롬복(@RequiredArgsConstructor)을 쓰거나, 그냥 생성자만 하나 만들어두면 알아서 주입됨.
```

---

## RowMapper 심층 분석

### RowMapper의 정의와 제네릭(Generic)
`RowMapper<T>`는 **DB의 테이블 데이터(`ResultSet`)를 자바 객체(`T`)로 변환**하는 매핑 전략(Strategy) 인터페이스이다.

* **`<T>`의 의미:** 이 매퍼가 최종적으로 반환할 객체의 타입을 의미한다.
    * `RowMapper<Member>`: DB 데이터를 읽어 **Member 객체**를 생성하는 매퍼.
* **데이터 저장 여부:** 
  * `RowMapper` 자체는 데이터를 저장하지 않는다. 
  * 단지 **"어떻게 변환할지"에 대한 로직(방법)** 만 가지고 있는 도구이다.

### 코드 비교: 익명 클래스 vs 람다식
가장 많이 사용하는 패턴이므로 문법 변화를 이해하는 것이 중요하다.

#### 1. 과거 방식 (익명 내부 클래스)
인터페이스를 직접 구현하여 객체를 생성하는 정석적인 방법이다.
```java
RowMapper<Member> memberRowMapper = new RowMapper<Member>() {
    @Override
    public Member mapRow(ResultSet rs, int rowNum) throws SQLException {
        Member member = new Member();
        member.setId(rs.getLong("id"));
        member.setName(rs.getString("name"));
        return member;
    }
};
```

#### 2. 현대 방식 (람다식)
불필요한 선언부를 제거하고 핵심 로직만 남긴 형태이다.
```java
// (매개변수) -> { 구현부 }
RowMapper<Member> memberRowMapper = (rs, rowNum) -> {
    Member member = new Member();
    member.setId(rs.getLong("id"));
    member.setName(rs.getString("name"));
    return member;
};
```

### 파라미터 상세 분석
위 람다식 `(rs, rowNum)`으로 넘어오는 두 인자에 대한 설명이다.

**1. `rs` (ResultSet)**
* **역할:** 현재 커서가 가리키고 있는 **DB의 한 행(Row)** 데이터다.
* **비유:**  LinkedList의 노드 하나, 혹은 엑셀의 가로 한 줄을 보고 있다고 생각하면 된다.
* **사용법:**
    * `rs.getString("name")`: 현재 행의 'name' 컬럼 값을 문자열로 꺼냄.
    * `rs.getLong("id")`: 현재 행의 'id' 컬럼 값을 Long 숫자로 꺼냄.
* **주의:** 
  * 람다 함수 내에서 `rs.next()`는 호출하지 않도록 한다. 
  * `JdbcTemplate`이 이미 커서를 이동시킨 상태로 `rs`를 넘겨주기 때문이다.

**2. `rowNum` (int)**
* **역할:** 현재 몇 번째 줄을 처리하고 있는지 알려주는 **행 번호(Index)** 이다. (0부터 시작)
* **용도:** 단순한 객체 매핑에서는 거의 사용하지 않으나, 인터페이스 규격상 반드시 선언해야 한다.

### 내부 동작 원리 (데이터 흐름)
> **Q. RowMapper는 몇 번째 줄에 어떤 데이터가 있는지 어떻게 아는가?**

- 앞서 말했듯 `RowMapper` 자체는 데이터를 기억(저장)하지 않는다. 
- **기억하고 반복하는 작업은 `JdbcTemplate`이 수행**하고, `RowMapper`는 매 순간 호출되어 변환만 수행한다.

**[JdbcTemplate 내부 동작 의사 코드 (Pseudo-code)]**

```java
// 개발자가 호출하는 시점: template.query("select...", memberRowMapper);

public List<Member> query(String sql, RowMapper<Member> rowMapper) {
    // 1. DB 조회
    ResultSet rs = connection.prepareStatement(sql).executeQuery();
    
    List<Member> results = new ArrayList<>(); // 최종 결과를 담을 리스트
    int rowNum = 0;
    
    // 2. 반복문 (JdbcTemplate이 수행)
    while (rs.next()) {
        // 3. 매퍼 호출 (여기서 우리가 만든 람다식이 실행됨)
        // "지금 커서(rs)가 가리키는 데이터를 Member 객체로 바꿔줘!"
        Member member = rowMapper.mapRow(rs, rowNum++);
        
        // 4. 리스트에 저장
        results.add(member);
    }
    
    return results; // 최종 리스트 반환
}
```
---

## SimpleJdbcInsert

### 개념
`INSERT` SQL 쿼리(`INSERT INTO table ...`)를 직접 작성하지 않고, 메서드 호출만으로 데이터를 저장하는 기능이다.

### 사용법 요약
**"설정하고 -> 맵에 담아서 -> 실행한다"** 흐름으로 동작한다.

```java
// 1. 설정 (생성 시점에 한 번)
SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate)
    .withTableName("member")            // 데이터를 넣을 테이블 이름
    .usingGeneratedKeyColumns("id");    // DB가 자동으로 생성해주는 키(PK) 컬럼명

// 2. 데이터 준비 (Map 사용)
Map<String, Object> parameters = new HashMap<>();
parameters.put("name", member.getName()); // 컬럼명 "name"에 값 매핑

// 3. 실행 및 키 받기
// executeAndReturnKey: 데이터를 넣고, 방금 생성된 PK값(id)을 바로 리턴해줌
Number key = jdbcInsert.executeAndReturnKey(new MapSqlParameterSource(parameters));
member.setId(key.longValue()); // 객체에 ID 업데이트
```

---

## JPA (Java Persistence API) 개요

### 1. 배경 및 필요성 (Why JPA?)
* **JdbcTemplate의 한계:** 반복 코드는 줄였지만, **여전히 SQL을 개발자가 직접 작성**해야 함.
* **패러다임의 불일치:** 객체 지향 프로그래밍(OOP)과 관계형 데이터베이스(RDB)는 근본적으로 다름.
    * 개발자가 중간에서 SQL을 매번 짜서 변환하는 "SQL 노가다"가 발생함.
* **JPA의 해결책:**
    * **객체 중심 설계:** SQL 중심의 개발에서 객체 중심의 설계로 패러다임 전환.
    * **생산성 향상:** 마치 자바 컬렉션(List)에 객체를 넣듯 저장하면, JPA가 알아서 SQL을 생성하고 실행함.

### 2. SQL Mapper vs ORM 비교
> "SQL Mapper와 ORM이 정확히 뭐지?"

| 구분 | 기술 예시 | 특징 |
| :--- | :--- | :--- |
| **SQL Mapper** | JdbcTemplate, MyBatis | • 개발자가 **SQL을 직접 작성**해야 함.<br>• SQL 실행 결과와 객체의 필드를 매핑하는 역할에 집중. |
| **ORM** | **JPA**, Hibernate | • **Object-Relational Mapping (객체-관계 매핑)**<br>• SQL을 작성하지 않음. **객체(Entity)와 DB 테이블을 매핑**하면, 프레임워크가 SQL을 대신 생성해줌. |

### 3. JPA와 Hibernate의 관계
* **JPA (Interface):** 자바 진영의 표준 인터페이스 (껍데기).
* **Hibernate (Implementation):** JPA 인터페이스를 실제로 구현한 라이브러리 (알맹이).
* **참고:** 이클립스 링크 등 다른 구현체도 있지만, 실무에서는 **거의 100% Hibernate**를 사용함.

---

## JPA 설정 및 사용법

### 1. 엔티티 매핑 (@Entity)
JPA가 관리할 객체임을 선언하는 어노테이션이다.
```java
@Entity // JPA가 관리하는 엔티티 선언
public class Member {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY) // PK 매핑 전략
    private Long id;
    
    private String name;
    // ... getter, setter
}
```

#### PK 생성 전략 (Identity Strategy)
* **Identity:** ID 값을 `null`로 넣으면, **DB가 알아서 AUTO_INCREMENT**로 값을 생성해주는 전략 (MySQL, H2 등).
* **기타:** Oracle의 `Sequence`, 테이블 채번 등이 있지만 주로 Identity를 많이 사용함.

### 2. EntityManager (핵심)
JPA의 모든 동작은 `EntityManager`를 통해서 이루어진다.

* **생성 및 주입 (Injection):**
    * 스프링 부트가 `application.properties` (DB 연결 정보) 등을 참고하여 **자동으로 EntityManager를 생성**하고 스프링 빈으로 등록해 둔다.
    * 개발자는 이를 주입받아 사용하면 된다.

#### [Deep Dive] 의존성 주입(DI) 조건
> **"주입해 줄 객체(Dependency)"와 "주입받을 객체(Client)" 모두 스프링 컨테이너에 빈(Bean)으로 등록되어 있어야 한다."**
>
> 스프링 컨테이너는 자신이 관리하는 객체들끼리만 서로를 연결(Injection)해 줄 수 있다. 컨테이너 바깥에 있는 객체(일반 자바 객체)는 스프링이 건드릴 수 없기 때문이다.
- `EntityManager`는 이미 스프링 빈으로 등록되어 있다. (스프링 부트에 의해 자동적으로)
- 이를 주입받아 사용하는 `MemberRepository` 같은 클래스도 `@Repository` 등을 붙여서 **스프링 빈으로 등록되어 있어야** DI가 가능하다.

---

## JPA 리포지토리 개발

### 1. 기본 CRUD (저장/조회)
`EntityManager` (`em`)가 제공하는 메서드를 사용한다.

```java
public class JpaMemberRepository implements MemberRepository {
    
    private final EntityManager em; // 주입 받음

    public JpaMemberRepository(EntityManager em) {
        this.em = em;
    }

    @Override
    public Member save(Member member) {
        em.persist(member); // 영구 저장 (INSERT SQL 실행)
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        Member member = em.find(Member.class, id); // PK로 조회 (SELECT SQL 실행)
        return Optional.ofNullable(member);
    }
}
```

### 2. JPQL (Java Persistence Query Language)
PK 조회가 아닌, **이름으로 조회(`findByName`)** 같은 복잡한 검색은 `JPQL`을 사용해야 한다.

* **정의:** **테이블이 아닌 엔티티 객체를 대상으로 하는 객체지향 쿼리.**
* **특징:**
    * SQL과 문법이 유사하지만, `FROM Member m` 처럼 **객체(Member)** 를 대상으로 함.
    * `SELECT m`은 해당 엔티티 객체 자체(모든 필드)를 조회한다는 의미.
    * 실행 시점에 JPA가 적절한 SQL로 번역하여 DB에 날림.

```java
public List<Member> findByName(String name) {
    // :name은 파라미터 바인딩을 의미
    return em.createQuery("select m from Member m where m.name = :name", Member.class)
             .setParameter("name", name) // 파라미터 값 설정
             .getResultList();
}
```

---

## 트랜잭션 (Transaction)

### 1. 주의사항 (필수)
> **"JPA의 모든 데이터 변경은 트랜잭션 안에서 실행되어야 한다."**

* 데이터를 저장(`persist`)하거나 변경할 때 트랜잭션이 없으면 예외가 발생하거나 반영되지 않는다.
* 보통 서비스 계층이나 리포지토리 메서드에 `@Transactional`을 붙여서 사용한다.

### 2. 테스트와 트랜잭션
* **`@Transactional` (테스트 코드):** 테스트 메서드에 붙이면, 테스트가 끝난 후 **자동으로 롤백(Rollback)** 하여 DB를 깔끔하게 유지해줌.
* **`@Commit`:** 롤백하지 않고 DB에 진짜로 반영하고 싶을 때 추가로 붙이는 어노테이션.

---

## 스프링 빈 등록 방식: 컴포넌트 스캔 vs 자바 설정 정보(@Bean)

### 핵심 결론
> **"클래스에 `@Component`가 없어도, `@Configuration` 설정 파일에서 `@Bean`으로 등록하면 스프링 빈이 된다."**

- 당초 가졌던 의문대로 `JpaMemberRepository` 코드만 보면 평범한 자바 클래스이다. 
- 하지만 `SpringConfig` 클래스에서 `new` 연산자로 직접 생성하여 반환했기 때문에, 스프링 컨테이너가 이 객체를 관리하게 된다.

---

### 동작 원리 상세 분석

#### 1. 상황 분석
* **`JpaMemberRepository`:** 클래스 위에 아무런 스프링 관련 어노테이션이 없음. (스캔 대상 아님)
* **`SpringConfig`:** `@Configuration`이 붙어있음. (스프링 설정 파일로 인식)

#### 2. 등록 과정 (수동 등록)
스프링 부트가 실행되면 다음과 같은 순서로 조립된다.

1.  **설정 로딩:** 스프링이 `@Configuration`이 붙은 `SpringConfig`를 읽어들인다.
2.  **메서드 실행:** `@Bean`이 붙은 `memberRepository()` 메서드를 실행한다.
3.  **객체 생성:** 메서드 내부의 `new JpaMemberRepository(entityManager)` 코드가 실행되면서 실제 객체가 메모리에 생성된다.
4.  **컨테이너 등록:** 생성된 객체(참조값)를 **'memberRepository'라는 이름의 빈(Bean)** 으로 컨테이너에 보관한다.

#### 3. EntityManager의 주입 흐름
그렇다면 `SpringConfig`는 `entityManager`를 어디서 가져왔을까?

```java
@Configuration
public class SpringConfig {

    // 1. 스프링 부트가 만들어준 EntityManager를 주입받음
    private final EntityManager em;

    @Autowired
    public SpringConfig(EntityManager em) {
        this.em = em;
    }

    @Bean
    public MemberRepository memberRepository() {
        // 2. 주입받은 em을 생성자에 넘겨주며 수동으로 빈 등록
        return new JpaMemberRepository(em);
    }
}
```

* **Step 1:** 스프링 부트가 **라이브러리 의존성(`build.gradle`)을 감지하여** `EntityManager`를 생성함. 
  * 이때 `application.properties`의 설정 값(DB 정보)들을 참조함(주입 받음).
* **Step 2:** `SpringConfig`도 스프링 빈이므로, 생성자를 통해 `EntityManager`를 주입받음.
* **Step 3:** `memberRepository()` 메서드에서 `new JpaMemberRepository(em)`을 호출할 때 이 `em`을 넘겨줌.
* **Step 4:** 결과적으로 `JpaMemberRepository`는 `EntityManager`를 가진 채로 생성되어 컨테이너에 들어감.

---

### 비교: 컴포넌트 스캔 vs @Bean 직접 등록

| 구분 | 컴포넌트 스캔 (자동) | 자바 코드 @Bean (수동) |
| :--- | :--- | :--- |
| **방법** | 클래스 위에 `@Component`, `@Service`, `@Repository` 붙이기. | 설정 파일(`Config`)에서 `@Bean` 메서드 작성하기. |
| **장점** | 개발이 편리하고 코드가 짧아짐. (실무 기본값) | **구현체를 변경할 때 코드를 손대지 않고 설정만 바꾸면 됨.** |
| **활용** | 일반적인 비즈니스 로직 (Controller, Service 등) | **라이브러리 설정**, **다형성을 적극 활용하는 경우** (현재 상황) |

> 지금처럼 `MemoryMemberRepository`를 썼다가, `JdbcTemplate`으로 바꿨다가, 다시 `JPA`로 바꾸는 상황에서는 **컴포넌트 스캔보다 자바 설정 정보(`@Bean`)를 사용하는 것이 좋다.**
>
> 왜냐하면 기존 코드를 전혀 손대지 않고, `SpringConfig` 파일에서 `return new ...` 부분만 딱 고치면 구현 기술이 교체되기 때문이다. (OCP 준수)

---

## IntelliJ 단축키 (Windows/Linux 기준)

상황이나 키맵 설정에 따라 다를 수 있으나, **IntelliJ의 공식 기본값(Default Keymap)** 은 다음과 같다.

| 기능 | 단축키 (Default) | 설명 |
| :--- | :--- | :--- |
| **Inline Variable** | `Ctrl` + `Alt` + `N` | 변수를 합쳐서 라인을 줄임. (Refactor 메뉴의 일부) |
| **Refactor This** | `Ctrl` + `Alt` + `Shift` + `T` | 리팩토링 메뉴 전체 보기 (여기서 Inline 선택 가능) |
| **Extract Method** | `Ctrl` + `Alt` + `M` | 코드를 메서드로 추출함. |

> 단축키가 안 먹힌다면 `File > Settings > Keymap`에서 검색해보거나, `Help > Find Action (Ctrl+Shift+A)`에서 기능 이름을 검색해보도록 하자.

---

## EntityManager 자동 등록 과정

스프링 부트는 **설정 파일(`application.properties`)이 아니라, 라이브러리 의존성(`build.gradle`)을 보고 빈을 등록**한다.

1.  **의존성 확인:** 프로젝트 시작 시, 스프링 부트는 `build.gradle`에 `spring-boot-starter-data-jpa` 라이브러리가 있는지 확인한다.
2.  **자동 설정 발동:** 라이브러리가 존재하면, 내부의 `JpaBaseConfiguration` 클래스가 실행된다.
3.  **빈 생성:** 이 설정 클래스가 `EntityManagerFactory`와 `EntityManager`를 자동으로 생성하고 스프링 빈으로 등록해 준다.
    * 이때 DB 연결 정보(`spring.datasource.url` 등)를 참고한다.

> 즉, `spring-boot-starter-data-jpa` 라이브러리만 추가하면 알아서 등록된다.

---

## application.properties 옵션 분석

### 1. SQL 로그 출력
```properties
spring.jpa.show-sql=true
```
* **기능:** JPA가 생성해서 실행하는 SQL을 콘솔창에 출력한다.
* **용도:** 개발 단계에서 내가 짠 코드가 어떤 쿼리로 변환되는지 눈으로 확인하기 위해 필수이다. 
  * 운영 환경에서는 보통 끈다.

### 2. DDL 자동 생성 전략 (ddl-auto)
```properties
spring.jpa.hibernate.ddl-auto=none
```
* **기능:** 애플리케이션 실행 시점에 테이블(DDL)을 어떻게 할 것인지 결정한다.
* **옵션 종류:**
    * **`create`:** 기존 테이블을 다 지우고(`DROP`) 새로 생성(`CREATE`). (개발 초기/테스트용)
    * **`create-drop`:** `create`와 같으나, 종료 시점에 테이블을 다 지움. (테스트용)
    * **`update`:** 변경된 스키마만 반영함. (DB에 데이터가 남음, 개발용)
    * **`validate`:** 엔티티와 테이블이 정상 매핑되었는지만 확인. (다르면 에러)
    * **`none`:** 아무것도 안 함. (자동 기능 끔)

> **주의:**
> 
> **운영 서버(Production)에서는 절대로 `create`, `create-drop`, `update`를 사용하면 안 된다.**
> 
> 실수로 `create`를 켜놓고 배포하면, **서버 띄우는 순간 고객 데이터가 전부 날아가기(DROP) 때문이다.** 그래서 실무에서는 보통 `none`이나 `validate`를 사용한다.

---

## 스프링 데이터 JPA (Spring Data JPA)

### 개요
JPA를 더 편리하게 사용하기 위해 스프링 프레임워크가 제공하는 래퍼(Wrapper) 기술이다.
> **핵심:** CRUD 메서드(저장, 조회, 수정, 삭제)를 반복해서 작성할 필요 없이, **인터페이스만 만들면 구현체를 자동으로 만들어준다.**

---

## 인터페이스 상속과 설정

### 1. 상속 문법 (extends)
* **규칙:** 인터페이스가 다른 인터페이스를 상속받을 때는 `implements`가 아니라 `extends`를 사용한다.
* **다중 상속:** 자바에서 클래스는 다중 상속이 불가능하지만, **인터페이스는 다중 상속이 가능**하다.

### 2. JpaRepository 상속
```java
// <T, ID> -> <엔티티 클래스, PK의 타입>
public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long>, MemberRepository {
    // 구현 코드 없음. 그냥 비어있음.
}
```
* `JpaRepository`를 상속받는 순간, 스프링 데이터 JPA가 "아, 이건 내가 관리해야겠다"라고 인식한다.
* `save()`, `findAll()`, `findById()` 같은 기본 CRUD 메서드가 자동으로 제공된다.

---

## 스프링 데이터 JPA 동작 원리

### 질문
> "인터페이스만 있는데 어떻게 동작하지? 구현체는 어쩌고?"

### 답변: 프록시(Proxy) 기술
스프링 부트가 애플리케이션을 로딩할 때 내부적으로 다음과 같은 일이 일어난다.

1.  **스캔:** `JpaRepository`를 상속받은 인터페이스(`SpringDataJpaMemberRepository`)를 발견한다.
2.  **프록시 생성:** 자바의 **리플렉션(Reflection)** 기술을 이용해 해당 인터페이스를 분석하고, 가짜 구현 객체(**프록시**)를 메모리에 찍어낸다.
    * *리플렉션:* 실행 중에 클래스나 메서드의 정보(이름, 타입 등)를 뜯어보는 기술.
3.  **빈 등록:** 이렇게 만들어진 프록시 객체를 스프링 컨테이너에 빈으로 등록한다.
4.  **주입:** 우리가 `MemberService`에서 주입받는 것은 인터페이스가 아니라, 스프링이 몰래 만든 **프록시 구현체**이다.

---

## 쿼리 메서드 (Query Methods)

### 메서드 이름으로 쿼리 생성
스프링 데이터 JPA는 메서드 이름을 분석해서 **JPQL(SQL)을 자동으로 생성**해준다. 이를 **쿼리 메서드** 기능이라고 한다.

### 예시
```java
// 메서드 이름만으로 SQL이 뚝딱 만들어짐
List<Member> findByName(String name);
// -> SELECT * FROM member WHERE name = ?

List<Member> findByNameAndId(String name, Long id);
// -> SELECT * FROM member WHERE name = ? AND id = ?
```

---

## 주요 기능 용어 정리

### 1. 페이징 (Paging)
* **개념:** 게시판 등에서 데이터를 한 번에 수천 개씩 가져오지 않고, **1페이지(10개), 2페이지(10개)...** 처럼 잘라서 가져오는 기능이다.
* **편의성:** 순수 DB 기술로는 `limit`, `offset` 계산 등 쿼리가 매우 복잡하지만, 스프링 데이터 JPA는 `Pageable` 인터페이스를 넘기면 알아서 계산해준다.

### 2. 동적 쿼리 (Dynamic Query)
* **개념:** 사용자의 검색 조건에 따라 SQL이 실시간으로 변하는 쿼리이다.
    * 이름만 검색할 때: `WHERE name = ?`
    * 이름과 나이를 검색할 때: `WHERE name = ? AND age = ?`
    * 아무것도 안 넣으면: `WHERE` 절 없음
* **문제점:** 기본 JPA나 스프링 데이터 JPA만으로는 `if`문을 써서 문자열을 더하는 등 처리가 복잡해진다.

---

## 동적 쿼리 vs 정적 쿼리

### 1. 핵심 결론
> **"입력값(파라미터)만 바뀌는 것은 동적 쿼리가 아니다."**

* **정적 쿼리 (Static Query):** SQL의 **문장 구조(뼈대)** 는 고정되어 있고, 그 안의 **데이터(`?`)** 만 바뀌는 경우.
* **동적 쿼리 (Dynamic Query):** 사용자의 입력 조건에 따라 **SQL 문장 자체(뼈대)** 가 늘어났다 줄어들었다 하는 경우.
---

### 2. 코드 비교

#### 상황 A: 정적 쿼리 (Parameter Binding)
사용자가 검색창에 "홍길동"을 입력하든 "김철수"를 입력하든, 실행되는 SQL의 형태는 **항상 똑같다.**

```sql
-- 입력: "홍길동"
SELECT * FROM member WHERE name = '홍길동';

-- 입력: "김철수"
SELECT * FROM member WHERE name = '김철수';
```

* **특징:** `WHERE name = ?` 라는 구조는 절대 변하지 않는다.
* **분류:** 이건 **정적 쿼리**이다. (대부분의 기본 조회)

#### 상황 B: 동적 쿼리 (Dynamic SQL)
쇼핑몰 검색 필터를 생각해보자.
1.  이름만 검색할 때
2.  이름 + 가격 범위를 같이 검색할 때

```sql
-- 1. 이름만 검색 (조건 1개)
SELECT * FROM member WHERE name = 'itemA';

-- 2. 이름 + 가격 검색 (조건 2개, SQL이 길어짐!)
SELECT * FROM member WHERE name = 'itemA' AND price > 10000;
```

* **특징:** 사용자가 "가격 필터"를 켰냐 안 켰냐에 따라 `AND price > 10000`이라는 **문장이 추가되거나 사라진다.**
* **분류:** SQL의 생김새 자체가 변하므로 **동적 쿼리**이다.

---

### 3. 왜 동적 쿼리가 어려운가? (Java 코드로 볼 때)

정적 쿼리는 그냥 문자열 하나(`String sql = "select..."`)로 끝난다. 하지만 동적 쿼리는 자바 코드로 짜려면 **지옥의 `if`문**이 필요하다.

```java
String sql = "SELECT * FROM member WHERE 1=1";

if (name != null) {
    sql += " AND name = ?"; // 문장을 이어 붙임
}

if (age != null) {
    sql += " AND age = ?"; // 문장을 또 이어 붙임
}

// 띄어쓰기 실수하기 딱 좋음 ("AND" 앞에 공백 안 넣어서 에러 나는 등)
```

### 결론
* **값만 바뀜:** 정적 쿼리 (JPA 기본 메서드로 해결 가능)
* **`AND` 조건이 붙었다 떨어졌다 함:** 동적 쿼리 (QueryDSL이 필요한 이유)

## 실무 기술 스택 조합 (Best Practice)

1.  **기본:** **JPA** + **스프링 데이터 JPA** (기본 CRUD 및 단순 조회 해결)
2.  **복잡한 쿼리:** **QueryDSL**
    * 자바 코드로 쿼리를 작성하게 해주는 기술.
    * 컴파일 시점에 오타를 잡아주고, **동적 쿼리를 매우 깔끔하게 작성 가능.**
3.  **해결 불가:** **Native Query** 또는 **JdbcTemplate**
    * 정말 복잡한 통계성 쿼리나 특정 DB 전용 기능이 필요할 때 사용.

---

## 스프링 데이터 JPA 인터페이스 상속 계층도

> 우리가 사용하는 `JpaRepository`는 사실 **최하위 자식**이다.

### 계층 구조 (Genealogy)

```text
Repository (최상위: 마커 인터페이스)
    ⬆️
CrudRepository (기본 CRUD 기능)
    ⬆️
PagingAndSortingRepository (페이징 + 정렬)
    ⬆️
JpaRepository (JPA 전용 기능 + 위 모든 기능)
```

---

### 각 인터페이스의 역할

### 1. Repository (Marker Interface)
* **역할:** 아무런 기능이 없다.
* **의의:** "이 인터페이스는 스프링 데이터 JPA가 관리하는 리포지토리입니다"라고 표시해 주는 **명찰(Marker)** 역할만 수행한다.

### 2. CrudRepository
* **역할:** 가장 기본적인 **CRUD(생성, 조회, 수정, 삭제)** 기능을 제공한다.
* **주요 메서드:**
    * `save(S entity)`
    * `findById(ID id)`
    * `count()`
    * `delete(T entity)`

### 3. PagingAndSortingRepository
* **역할:** 데이터를 잘라서 가져오거나(Paging), 순서대로 정렬(Sorting)하는 기능을 추가한다.
* **주요 메서드:**
    * `findAll(Sort sort)`
    * `findAll(Pageable pageable)`

### 4. JpaRepository (우리가 쓰는 것)
* **역할:** 위 3가지를 모두 상속받고, 추가로 **JPA에 특화된 기능**을 제공한다.
* **추가된 기능:**
    * `flush()`: 영속성 컨텍스트의 변경 내용을 DB에 즉시 반영.
    * `saveAndFlush()`: 저장하고 바로 플러시.
    * `deleteInBatch()`: 여러 개를 한 번에 삭제 (성능 최적화).

---

### 결론: 왜 실무에서는 JpaRepository를 쓰는가?

```java
// 실무 코드 예시
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

우리가 굳이 부모 인터페이스(`CrudRepository`)를 쓰지 않고 **`JpaRepository`를 상속받는 이유**는 단순하다.

1.  **편의성:** CRUD, 페이징, JPA 최적화 기능까지 **모두 포함된 풀옵션**이기 때문이다.
2.  **다형성:** `JpaRepository` 타입으로 받으면, 필요할 때 페이징 기능이나 배치 삭제 기능을 바로 사용할 수 있다.

> 입문 단계에서는 `JpaRepository`가 다 가지고 있다는 것만 알면 된다. 나중에 기능을 제한하고 싶을 때 상위 인터페이스를 쓰기도 하지만, 99%의 상황에서는 그냥 `JpaRepository`를 쓰면 된다."

---

## 영속성 컨텍스트 (Persistence Context)

### 정의
**"엔티티(Entity)를 영구 저장하는 환경"** 이라는 뜻의 논리적인 개념이다.

* **위치:** `EntityManager` 안에 눈에 보이지 않는 **가상의 저장소(공간)** 가 있다고 상상하면 된다.
* **역할:** 애플리케이션(Java)과 데이터베이스(DB) 사이에서 **중간 다리(버퍼)** 역할을 한다.
* **핵심:** 우리가 `em.persist(member)`를 하면, DB에 바로 `INSERT` SQL을 날리는 것이 아니라, 일단 이 **영속성 컨텍스트에 저장**한다.

---

## 영속성 컨텍스트가 주는 4가지 이점

JPA를 쓰면 성능이 느려질 것 같지만, 이 중간 계층 덕분에 오히려 성능을 최적화할 수 있는 여지가 생긴다.

### 1. 1차 캐시 (First Level Cache)
영속성 컨텍스트 내부에는 `Map<KEY, VALUE>` 형태의 캐시가 있다.

* **동작:**
    1.  `em.find(Member.class, "member1")` 호출.
    2.  DB에 가기 전에 먼저 **1차 캐시**를 뒤진다.
    3.  있으면? -> **DB 조회 안 하고** 캐시에서 바로 반환. (속도 빠름)
    4.  없으면? -> DB 조회 후, 1차 캐시에 저장하고 반환.
* **특징:** 트랜잭션이 끝나면 캐시도 다 날아간다. (아주 짧은 시간 동안만 유지되는 캐시)

### 2. 동일성 보장 (Identity)
자바 컬렉션에서 꺼낸 객체처럼, 같은 ID로 조회한 엔티티는 **`==` 비교 시 `true`**가 나온다.

```java
Member a = em.find(Member.class, "member1");
Member b = em.find(Member.class, "member1");

System.out.println(a == b); // true (참조값 주소가 완전히 같음)
```

### 3. 트랜잭션을 지원하는 쓰기 지연 (Transactional Write-behind)
`em.persist()`를 할 때마다 SQL을 날리는 게 아니다.

1.  `em.persist(memberA)` -> 1차 캐시에 넣고, SQL은 **'쓰기 지연 SQL 저장소'** 에 쌓아둔다.
2.  `em.persist(memberB)` -> 역시 저장소에 쌓아둔다. (아직 DB 전송 X)
3.  `transaction.commit()` -> **이 시점에** 쌓여있던 SQL들을 한방에 DB로 보낸다 (`flush`).

### 4. 변경 감지 (Dirty Checking) - ★
> **Q. 왜 JPA에는 `update()` 메서드가 없을까?**
>
> **A. 자바 컬렉션처럼 다루기 때문이다. 리스트에서 객체 꺼내서 값 바꿨다고 다시 `list.add()` 안 하잖아?**

**원리:**
1.  JPA는 트랜잭션이 커밋되는 시점에, 엔티티와 **스냅샷(처음 읽어왔을 때의 상태)** 을 비교한다.
2.  "어? 이름(name) 필드가 바뀌었네?"
3.  **알아서 `UPDATE` SQL을 생성**하여 쓰기 지연 저장소에 등록한다.
4.  DB에 반영한다.

```java
// [수정 코드 예시]
Member member = em.find(Member.class, "member1");

// 데이터 수정
member.setName("ZZANGGU");

// em.update(member) 같은 코드가 필요 없음! (자동 감지)
transaction.commit(); 
```

---

## 엔티티의 생명주기 (Entity Lifecycle)

객체가 영속성 컨텍스트와 어떤 관계를 맺고 있는지에 따라 상태가 나뉜다.

1.  **비영속 (New):** 객체를 생성만 한 상태. (`new Member()`)
2.  **영속 (Managed):** `em.persist(member)`를 통해 영속성 컨텍스트에 들어간 상태. (이제부터 JPA가 관리함)
3.  **준영속 (Detached):** 관리되다가 쫓겨난 상태. (`em.detach(member)`)
4.  **삭제 (Removed):** 삭제된 상태. (`em.remove(member)`)

> 영속성 컨텍스트는 **'엔티티를 영원히 저장하는 환경'** 이지만, 실제로는 트랜잭션이라는 단위 안에서만 동작한다. 
> 
> 즉, **'트랜잭션을 시작할 때 생성되고, 끝날 때 종료된다'** 고 이해하면 된다.