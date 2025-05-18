# JDBC 트랜잭션 이해

---

## ✅ 학습 주제

* 트랜잭션 개념 및 ACID 원칙
* 자동 커밋 vs 수동 커밋 차이점
* 데이터베이스 세션, 커넥션, 락 동작 구조
* 트랜잭션 적용 방식 및 커넥션 유지 전략

---

## 🧩 핵심 개념 요약

| 항목                | 설명                                         |
| ----------------- | ------------------------------------------ |
| 트랜잭션(Transaction) | 논리적 작업 단위. 모두 성공하거나 모두 실패해야 하는 원자적 연산 집합   |
| 커밋(Commit)        | 트랜잭션 결과를 실제 DB에 반영                         |
| 롤백(Rollback)      | 트랜잭션을 취소하고 이전 상태로 복구                       |
| 자동 커밋             | SQL 실행 직후 자동으로 커밋. 트랜잭션 제어 불가              |
| 수동 커밋             | 직접 `commit()` 또는 `rollback()`을 호출해야 결과 반영됨 |
| 세션(Session)       | 커넥션 단위로 DB 내부에서 관리되는 상태. 트랜잭션은 세션과 연결됨     |

---

## 🔐 트랜잭션 ACID 원칙

| 원칙                | 설명                            |
| ----------------- | ----------------------------- |
| 원자성 (Atomicity)   | 트랜잭션 내 작업은 전부 성공하거나 전부 실패해야 함 |
| 일관성 (Consistency) | 트랜잭션 전후 데이터 무결성이 유지되어야 함      |
| 격리성 (Isolation)   | 동시에 실행되는 트랜잭션 간 간섭 방지         |
| 지속성 (Durability)  | 커밋된 데이터는 시스템 오류 후에도 유지됨       |

---

## 🛠️ 트랜잭션 제어 실습 흐름

### 1. 자동 커밋 모드

```sql
set autocommit true;
insert into member values ('data1', 10000); -- 자동 커밋됨
```

### 2. 수동 커밋 모드

```sql
set autocommit false;
insert into member values ('data2', 10000);
commit;  -- 명시적으로 커밋
```

* 수동 커밋 모드에서 커밋 전까지는 **다른 세션에서는 변경 내용이 보이지 않음**

---

## 💥 계좌이체 예시 시나리오

### 정상 흐름

```sql
set autocommit false;
update member set money = money - 2000 where member_id = 'memberA';
update member set money = money + 2000 where member_id = 'memberB';
commit;
```

### 오류 발생 → 커밋

```sql
set autocommit false;
update member set money = money - 2000 where member_id = 'memberA';
update member set money = money + 2000 where member_iddd = 'memberB'; -- 오류
commit; -- ❌ 위험! A는 돈을 잃고 B는 못 받음
```

### 오류 발생 → 롤백

```sql
rollback; -- 🔁 이전 상태로 복구
```

---

## 🧱 락(Lock)과 동시성 제어

| 상황                  | 설명                       |
| ------------------- | ------------------------ |
| 변경 중 다른 세션이 같은 행 수정 | 락을 획득한 세션이 커밋 전까지 대기 상태  |
| 락 타임아웃              | 설정된 시간 내 락 획득 실패 시 오류 발생 |
| SELECT FOR UPDATE   | 조회 시점에 락을 걸어 이후 변경 방지 가능 |

---

## 💼 애플리케이션 트랜잭션 적용 전략

### MemberServiceV2: JDBC 수동 트랜잭션

```java
con.setAutoCommit(false);  // 트랜잭션 시작
try {
    bizLogic(con);         // 비즈니스 로직 (Connection 유지)
    con.commit();          // 커밋
} catch (Exception e) {
    con.rollback();        // 롤백
    throw e;
} finally {
    con.setAutoCommit(true); // 커넥션 상태 복구 후 반환
    con.close();
}
```

* **Connection을 직접 파라미터로 전달**하여 트랜잭션 범위를 유지
* 비즈니스 로직과 트랜잭션 관리 코드를 분리하여 명확성 확보
* 트랜잭션을 시작한 계층이 **커밋, 롤백, 커넥션 정리까지 책임**져야 함

---

## ⚠️ 주의사항 및 실무 전략

| 항목         | 주의 내용                                    |
| ---------- | ---------------------------------------- |
| 커넥션 풀      | 커밋/롤백 없이 반환 시 트랜잭션이 열린 상태로 다른 사용자에게 전달됨  |
| 예외 발생 시 롤백 | 반드시 `try-catch`로 감싸서 롤백 실패까지 대비해야 함      |
| 락 충돌 방지    | 필요한 경우 `SELECT FOR UPDATE`를 활용해 변경 충돌 방지 |
| 트랜잭션 책임 위치 | 서비스 계층이 트랜잭션을 제어하고, DAO는 커넥션만 사용해야 함     |


---
## 🔍 con.close()의 의미

### 1. 커넥션 풀을 사용하지 않는 경우 (ex. 순수 DriverManager)

* `con.close()` → DB 커넥션이 실제로 종료됨
* 이후 사용 시 새 커넥션을 생성해야 함

### 2. 커넥션 풀을 사용하는 경우 (ex. HikariCP)

* `con.close()` → 커넥션이 **DB와의 연결을 끊는 게 아니라 풀에 반환**
* 다시 사용할 때 기존 커넥션을 재활용함

🚨 이때 문제:
트랜잭션을 위해 `setAutoCommit(false)`로 설정한 커넥션이
**autoCommit=false인 채로 풀에 반환되면**, 다음 사용자도 트랜잭션이 꺼진 커넥션을 받게 됨
→ **트랜잭션 누수**, **데이터 반영 안됨**, **예상치 못한 롤백 실패** 등 치명적 오류 발생

---

## ✅ 그래서 반드시 해야 하는 것

```java
con.setAutoCommit(true);
```

* 커넥션을 반환하기 전에 **기본 상태로 복구**
* **트랜잭션 종료 후 상태를 초기화하지 않으면**, 다음 사용자가 예상치 못한 상태로 커넥션을 받게 됨

---

## ✅ 결론

> `// 커넥션 풀 고려`는 커넥션을 **풀에 반환하기 전에 상태를 반드시 초기화**해야 함을 명확히 하기 위한 중요한 실무 주석.
> 특히 `setAutoCommit(false)` 설정을 썼다면 **반드시 true로 돌려놓고 close()** 해야 안전함


