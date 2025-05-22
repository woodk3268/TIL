# 자바 예외 이해

---

## ✅ 학습 주제

* 자바 예외 계층 및 체크/언체크 예외 구분
* 예외 처리 방식과 던지기 방식의 차이
* 예외 전환 전략과 포함(포장) 원칙
* 실무에서의 예외 선택 기준

---

## 🧩 자바 예외 계층 구조 요약

| 클래스                | 설명                                     |
| ------------------ | -------------------------------------- |
| `Throwable`        | 모든 예외와 오류의 최상위 클래스                     |
| `Error`            | 복구 불가능한 시스템 예외 (예: OOM, StackOverflow) |
| `Exception`        | 애플리케이션 로직용 예외. **체크 예외**               |
| `RuntimeException` | 실행 중 발생하는 예외. **언체크 예외**               |

> ☑️ 개발자는 `Exception`부터 시작하여 필요한 예외만 다루어야 하며 `Error`나 `Throwable`은 잡지 않는다.

---

## ⚙️ 체크 예외 vs 언체크 예외

| 구분     | 체크 예외                         | 언체크 예외                                             |
| ------ | ----------------------------- | -------------------------------------------------- |
| 기준     | `Exception`을 상속               | `RuntimeException`을 상속                             |
| 컴파일 체크 | 예외 처리 또는 `throws` 필수          | 생략 가능                                              |
| 용도     | 복구 가능한 예외 (예: 로그인 실패)         | 복구 불가능 or 시스템성 예외 (예: DB 오류)                       |
| 예      | `IOException`, `SQLException` | `NullPointerException`, `IllegalArgumentException` |

---

## 🔁 예외 처리 방식 2가지

### 1. 예외 잡아서 처리

```java
try {
    repository.call();
} catch (MyCheckedException e) {
    log.error("예외 처리", e);
}
```

### 2. 예외 던지기

```java
public void call() throws MyCheckedException {
    repository.call();
}
```

> ✅ 체크 예외는 반드시 둘 중 하나 선택해야 함
> ✅ 언체크 예외는 `throws` 생략 가능하지만 필요시 문서화 또는 명시 가능

---

## ⚠️ 체크 예외의 문제점

### ❌ 복구 불가능한 예외인데도 `throws` 선언 강제

```java
void logic() throws SQLException, ConnectException
```

* DB, 네트워크 오류는 대부분 복구 불가능
* 컨트롤러, 서비스 모두 예외를 처리하지 못하면서도 예외 타입을 선언해야 하는 **의존성 문제** 발생

### ❌ 기술 변경 시 파급 효과

* JDBC → JPA 변경 시 `SQLException` → `JPAException` 등 예외 전파 필요
* → 서비스, 컨트롤러가 전부 수정 대상

---

## ✅ 언체크 예외 활용 전략

### 실무에서는 대부분 언체크 예외 사용

* 복구 불가능한 예외를 `RuntimeException`으로 포장하여 던짐
* 서비스, 컨트롤러는 예외에 의존하지 않고 예외가 발생하면 \*\*공통 처리기 (e.g. `@ControllerAdvice`)\*\*가 처리

```java
try {
    runSQL();
} catch (SQLException e) {
    throw new RuntimeSQLException(e); // 기존 예외 포함!
}
```

> ☑️ 반드시 기존 예외를 포함해야 스택 트레이스 추적 가능

---

## 📦 예외 전환 시 스택 트레이스 포함

| 포함 방식                     | 효과               |
| ------------------------- | ---------------- |
| `new RuntimeException(e)` | 기존 예외 스택 유지 (추천) |
| `new RuntimeException()`  | 기존 예외 정보 손실 (지양) |

---

## 🧭 실무 원칙 정리

| 항목       | 원칙                                            |
| -------- | --------------------------------------------- |
| 기본 방침    | **언체크 예외를 기본으로 사용**                           |
| 체크 예외 사용 | 복구 가능한 예외에 한정하여 사용 (예: 포인트 부족 등 비즈니스 예외)      |
| 전환 전략    | 시스템 예외(예: `SQLException`) → 언체크 예외로 포장        |
| 문서화      | 중요한 언체크 예외는 `throws` 또는 JavaDoc으로 명시하여 의도 드러냄 |

---

## 🧠 인사이트

* 예외는 복구 가능 여부를 중심으로 선택해야 한다.
* 복구 불가능한 예외를 체크 예외로 만들면 **의존성 누수, 유지보수 비용 증가, 유연성 저하**가 발생한다.
* 예외를 명확히 전환하고 포함(log 포함 또는 cause 저장)하지 않으면 **디버깅 자체가 불가능**해진다.
* 결국 예외는 \*\*처리 주체가 있는가?\*\*를 기준으로 체크 여부를 판단해야 한다.

---

## 📦 예외 추상화와 디버깅을 동시에: Wrapper 패턴의 실전 활용
✅ Wrapper 패턴 (Exception Wrapping) 이란?

Wrapper 패턴은 **하위 계층에서 발생한 예외를 상위 계층에 전달할 때**, 그 예외를 **다른 타입의 예외로 감싸서 던지는 패턴**. 주로 checked 예외를 unchecked 예외로 감쌀 때 사용.

---

### 🔍 왜 Wrapper 패턴을 사용하는가?

1. **상위 계층이 예외에 직접적으로 의존하지 않도록 하기 위해**

   * 예: `SQLException`은 JDBC에만 있는 checked 예외.
   * 컨트롤러나 서비스 계층은 DB에 대한 세부사항을 몰라도 되므로, 직접 `SQLException`을 처리하게 해선 안 된다.

2. **서비스 계층에서 예외를 도메인 예외로 전환**

   * 예: `OrderRepository`에서 발생한 `SQLException`을 `OrderSaveException`으로 감싸서 던진다.

3. **다양한 예외를 하나의 계층에서 통합 관리하고, 일관된 예외 처리 전략 사용 가능**

---

### ⚙️ 구현 예시

```java
public class RuntimeSQLException extends RuntimeException {
    public RuntimeSQLException(Throwable cause) {
        super(cause); // 기존 예외 포함
    }
}
```

```java
public void call() {
    try {
        runSQL();
    } catch (SQLException e) {
        throw new RuntimeSQLException(e); // 래핑
    }
}
```

---

### ❗ 기존 예외 포함 여부의 차이

| 구분         | 기존 예외 포함                     | 기존 예외 미포함                   |
| ---------- | ---------------------------- | --------------------------- |
| **코드**     | `new RuntimeSQLException(e)` | `new RuntimeSQLException()` |
| **스택트레이스** | 모든 계층에서 발생한 예외 추적 가능         | 감싼 예외만 보여서 디버깅 불가           |
| **원인 추적**  | `Caused by:` 절로 원인 확인 가능     | 원인 알 수 없음                   |
| **실무 적합도** | ✅ 필수                         | ❌ 매우 위험                     |

---

### 📌 예외 포함 로그 출력 방식

```java
try {
    controller.request();
} catch (Exception e) {
    log.info("예외 발생", e);  // 스택 트레이스 포함됨
}
```
---

### 📎 스택 트레이스 비교

#### ✅ 기존 예외 포함

```
RuntimeSQLException: java.sql.SQLException: ex
  at Repository.call(...)
  ...
Caused by: java.sql.SQLException: ex
  at Repository.runSQL(...)
```

#### ❌ 기존 예외 미포함

```
RuntimeSQLException: null
  at Repository.call(...)
```

> "Caused by"가 없기 때문에 **실제 문제의 원인을 절대 찾을 수 없음**

---

### ☑️ 핵심 정리

* **예외 전환 시 반드시 원인을 포함 (`super(e)`)**
* **Wrapper 예외 클래스는 `Throwable`을 인자로 받는 생성자 제공**
* **기술적 세부사항은 숨기되, 디버깅은 가능하게 만들어야 함**
* **서비스/컨트롤러 계층에서는 도메인 친화적인 예외로 추상화**

