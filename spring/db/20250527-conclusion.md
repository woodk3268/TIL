# 데이터 접근 기술 - 활용 방안

---

## ✅ 학습 주제

* **DI/OCP 원칙** vs **실용적 구조** 사이의 트레이드오프 이해
* 스프링 데이터 JPA + QueryDSL 조합 방식
* 다양한 데이터 접근 기술의 조합 전략 (JPA + MyBatis 등)
* `JpaTransactionManager`를 통한 통합 트랜잭션 관리

---

## 🧩 핵심 개념 요약

| 항목                      | 설명                                |
| ----------------------- | --------------------------------- |
| 어댑터 패턴                  | 기존 인터페이스를 유지하면서 구현 기술을 변경할 수 있음   |
| 실용적 구조                  | 스프링 데이터 JPA와 QueryDSL 기능을 분리하여 사용 |
| `ItemRepositoryV2`      | CRUD 및 단순 조회: Spring Data JPA 담당  |
| `ItemQueryRepositoryV2` | 동적 복잡 조회: QueryDSL 담당             |
| `ItemServiceV2`         | 두 Repository를 직접 의존하여 서비스 구성      |
| `JpaTransactionManager` | JPA 기반 트랜잭션 관리기, JDBC도 함께 처리 가능   |

---

## ⚖️ 구조 설계 시 트레이드오프

| 선택                     | 장점                   | 단점                  |
| ---------------------- | -------------------- | ------------------- |
| **OCP/DI 유지 (어댑터 사용)** | 서비스 코드 불변, 구현체 교체 용이 | 구조 복잡, 코드 양 증가      |
| **서비스에서 직접 구현체 사용**    | 구조 단순, 개발 빠름         | OCP 원칙 위반, 유지보수 어려움 |

💡 추상화도 비용이다 → 팀/상황에 맞는 유연한 판단이 중요

---

## 🛠️ 실용적인 구조 (권장)

### 💡 기본 CRUD → `Spring Data JPA`

```java
public interface ItemRepositoryV2 extends JpaRepository<Item, Long> {}
```

### 💡 복잡한 조회 → `Querydsl`

```java
@Repository
public class ItemQueryRepositoryV2 {
    private final JPAQueryFactory query;

    public List<Item> findAll(ItemSearchCond cond) {
        return query.select(item)
                    .from(item)
                    .where(
                        likeItemName(cond.getItemName()),
                        maxPrice(cond.getMaxPrice()))
                    .fetch();
    }
}
```

### 💡 서비스 계층

```java
@Service
@Transactional
public class ItemServiceV2 implements ItemService {
    private final ItemRepositoryV2 jpaRepo;
    private final ItemQueryRepositoryV2 queryRepo;

    public List<Item> findItems(ItemSearchCond cond) {
        return queryRepo.findAll(cond);
    }
}
```

---

## 🔀 다양한 기술 조합 전략

| 기술                    | 역할              | 비고        |
| --------------------- | --------------- | --------- |
| JPA / Spring Data JPA | CRUD, 도메인 중심 접근 | 실무 기본     |
| QueryDSL              | 타입 안전 + 동적 쿼리   | 실무 필수     |
| JdbcTemplate          | 성능 민감한 단순 SQL   | 보조 용도     |
| MyBatis               | 복잡한 통계/다건 JOIN  | 일부 영역 보완용 |

---

## 🔄 트랜잭션 관리 전략

| 기술 조합      | 트랜잭션 매니저                       | 설명                    |
| ---------- | ------------------------------ | --------------------- |
| JPA + JDBC | `JpaTransactionManager`        | JDBC도 처리 가능           |
| JDBC only  | `DataSourceTransactionManager` | 별도 등록 필요 없음 (단일 사용 시) |

**주의:** JPA와 JDBC 혼합 시, **JPA의 flush 시점**에 주의
→ JDBC 이전에 `em.flush()` 호출 필요

---

## 💡 인사이트

* “모든 것을 추상화하려 하지 말고, 유지비용과 실용성을 고려해 설계”
* **기본은 JPA + Spring Data JPA + Querydsl 조합**으로 시작하고
  → 해결이 어려운 쿼리만 JDBC / MyBatis로 커버하는 방식이 실무에 적합
* 복잡도를 낮추고 유지보수성 높은 코드를 위해 **역할을 분리한 설계** 권장

---

## ✅ 데이터 접근 기술 요약표

| 기술                  | 설명                   | 추천 상황       |
| ------------------- | -------------------- | ----------- |
| **JdbcTemplate**    | SQL 직접 작성, 단순 API    | 매우 간단한 쿼리   |
| **MyBatis**         | SQL Mapper, 복잡 쿼리 적합 | 다건 조회/통계    |
| **JPA**             | ORM, 객체 중심 개발        | 표준 ORM      |
| **Spring Data JPA** | CRUD 자동화, 추상화 지원     | 단순 업무 CRUD  |
| **QueryDSL**        | 타입 안전 쿼리, 동적 조건      | 복잡/가변 조건 쿼리 |

---

필요하시면 위 내용을 기반으로 **종합 기술 선택 가이드**, **구조별 장단점 표**, 또는 **TIL 전체 PDF**로 정리해드릴 수도 있습니다.
