# 스프링과 예외 처리/반복 제거 전략

---

## ✅ 학습 주제

* JDBC 예외 누수 문제 해결 방법
* 체크 예외 → 런타임 예외 전환 전략
* 스프링 예외 추상화 적용 (`SQLExceptionTranslator`)
* `JdbcTemplate`을 활용한 반복 제거

---

## 🧩 주요 문제 및 해결 전략

| 문제                            | 스프링의 해결 방식                      |
| ----------------------------- | ------------------------------- |
| `SQLException` 누수 → 서비스 계층 오염 | 런타임 예외(Custom or Spring 제공)로 전환 |
| 인터페이스가 기술에 종속됨                | 예외 제거로 순수 인터페이스 유지              |
| DB 오류코드마다 직접 처리               | 스프링 예외 변환기로 자동 변환               |
| JDBC 반복 코드 (커넥션, close 등)     | `JdbcTemplate`으로 공통 처리          |

---

## 💣 체크 예외의 문제점

* 인터페이스가 `throws SQLException`을 포함하게 되며 JDBC 기술에 종속됨
* 기술 교체 시 인터페이스 전체 수정 필요
* 예외 복구가 어려움 (`SQLException` 내부를 직접 열어야 함)

---

## ✅ 런타임 예외 전환

```java
catch (SQLException e) {
    throw new MyDbException(e); // cause 포함 필수
}
```

* 예외 변환 시 기존 예외 반드시 포함 (스택 트레이스 추적 가능)
* `MyDbException`을 상속하여 `MyDuplicateKeyException`처럼 상황별 예외로 분기 가능

---

## 🧠 스프링 예외 추상화

| 예외 분류                 | 예시                                                          |
| --------------------- | ----------------------------------------------------------- |
| NonTransient (일시적 아님) | `BadSqlGrammarException`, `DataIntegrityViolationException` |
| Transient (일시적)       | `QueryTimeoutException`, `CannotAcquireLockException`       |

### 예외 변환기 사용법

```java
SQLExceptionTranslator translator = new SQLErrorCodeSQLExceptionTranslator(dataSource);
throw translator.translate("save", sql, e);
```

* 내부적으로 `sql-error-codes.xml`을 참고해 DB 벤더별 코드 매핑 수행
* ex) H2의 `23505` → `DuplicateKeyException`

---

## 🔁 반복 제거: JdbcTemplate

```java
JdbcTemplate template = new JdbcTemplate(dataSource);

template.update("insert into member(member_id, money) values(?, ?)", id, money);
```

* 커넥션 획득, 예외 변환, 자원 해제 자동 처리
* ResultSet 매핑은 `RowMapper<T>`를 통해 분리

```java
private RowMapper<Member> memberRowMapper() {
    return (rs, rowNum) -> new Member(rs.getString("member_id"), rs.getInt("money"));
}
```

---

## ✅ 완성 구조 요약

| 계층         | 내용                                            |
| ---------- | --------------------------------------------- |
| Repository | `MemberRepository` 인터페이스 기반, 구현은 기술에 따라 교체 가능 |
| Service    | 예외, 트랜잭션, 기술 종속 없는 순수 로직                      |
| 예외         | 스프링 예외 변환기 + 런타임 전환으로 기술 독립성 확보               |
| JDBC 코드    | `JdbcTemplate`으로 반복 제거, 예외 자동 변환              |

