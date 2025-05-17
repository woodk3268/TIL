# TIL - 스프링과 트랜잭션 문제 해결

---

## ✅ 학습 주제

* JDBC 기반 트랜잭션의 한계와 스프링의 해결책
* `PlatformTransactionManager`를 활용한 트랜잭션 추상화
* 커넥션 동기화 방식 (ThreadLocal 기반)
* `@Transactional` 기반 AOP 적용 전략

---

## 🧩 문제 인식 및 설계 원칙

| 문제점                | 설명                                          |
| ------------------ | ------------------------------------------- |
| 서비스 계층에 JDBC 기술 노출 | `Connection`, `SQLException` 등이 비즈니스 로직을 오염 |
| 트랜잭션 커넥션 공유 문제     | 수동 커밋을 위해 커넥션을 명시적으로 파라미터로 전달               |
| 중복 코드 문제           | `try-catch-finally`, 커밋/롤백 반복               |
| 예외 누수              | `SQLException`이 상위 계층으로 전달됨                 |

---

## ⚙️ 스프링의 해결 전략

### 1. 트랜잭션 추상화

* 트랜잭션 인터페이스: `PlatformTransactionManager`
* 구현체 예시: `DataSourceTransactionManager`, `JpaTransactionManager`
* JDBC/JPA 등 기술에 무관하게 통일된 API로 트랜잭션 시작/커밋/롤백 수행

### 2. 트랜잭션 동기화 (커넥션 공유)

* `TransactionSynchronizationManager`가 `ThreadLocal`을 활용하여 커넥션 저장
* DAO는 `DataSourceUtils.getConnection()`를 통해 트랜잭션과 동기화된 커넥션 획득

### 3. 트랜잭션 템플릿

* `TransactionTemplate.execute()` 메서드로 트랜잭션 처리 구조 간소화
* 반복되는 트랜잭션 로직을 제거하고 비즈니스 로직만 작성 가능

### 4. 트랜잭션 AOP (@Transactional)

* 서비스 계층에서 `@Transactional` 애노테이션으로 선언적 트랜잭션 적용
* 프록시를 통해 트랜잭션 시작, 커밋/롤백을 자동으로 처리
* 핵심 비즈니스 로직과 부가 기능(트랜잭션 처리) 완전 분리 가능

---

## 💡 주요 클래스 및 개념 요약

| 구성 요소                               | 설명                      |
| ----------------------------------- | ----------------------- |
| `PlatformTransactionManager`        | 트랜잭션 시작, 커밋, 롤백 API 추상화 |
| `DataSourceTransactionManager`      | JDBC 기반 트랜잭션 매니저        |
| `TransactionTemplate`               | 템플릿 콜백 기반 트랜잭션 처리 도구    |
| `@Transactional`                    | 선언적 트랜잭션 처리 방식 (AOP 기반) |
| `TransactionSynchronizationManager` | 쓰레드 기반 커넥션 동기화 관리기      |
| `DataSourceUtils.getConnection()`   | 현재 쓰레드의 트랜잭션 커넥션 획득     |

---

## 🔁 비교 정리

| 방식                       | 장점               | 단점                 |
| ------------------------ | ---------------- | ------------------ |
| 수동 JDBC 트랜잭션             | 제어권 명확           | 기술 종속 심함, 코드 중복    |
| TransactionManager 수동 사용 | 추상화 도입           | 커밋/롤백 중복 여전        |
| TransactionTemplate      | 중복 제거, 명확한 구조    | 여전히 트랜잭션 처리 코드 존재  |
| @Transactional           | 선언만으로 적용, AOP 활용 | 세밀한 제어에는 부적합할 수 있음 |

---

## 🧪 테스트 사례 요약

* 정상 이체: 트랜잭션 커밋 → 금액 정상 반영
* 예외 상황: 트랜잭션 롤백 → 금액 원상복귀
* AOP 프록시 확인: 서비스 객체는 프록시 적용 (`CGLIB`), Repository는 적용되지 않음

---

## ⚙️ 스프링 부트의 자동 설정

| 자동 등록 대상                     | 설명                                             |
| ---------------------------- | ---------------------------------------------- |
| `DataSource`                 | `application.properties`의 설정값으로 자동 등록          |
| `PlatformTransactionManager` | JDBC 사용 시 `DataSourceTransactionManager` 자동 등록 |
| 개발자 수동 등록 시                  | 스프링 부트 자동 설정은 동작하지 않음                          |

