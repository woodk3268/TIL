# 데이터 접근 기술 - 스프링 JdbcTemplate

---

## ✅ 학습 주제

* JdbcTemplate의 구조와 동작 방식
* 순서 기반 → 이름 기반 파라미터 바인딩으로의 발전
* 동적 쿼리 처리 방식 및 한계
* SimpleJdbcInsert의 편의 기능 활용

---

## 🧩 핵심 개념 요약

| 개념                           | 설명                                                                    |
| ---------------------------- | --------------------------------------------------------------------- |
| `JdbcTemplate`               | 순서 기반 파라미터 바인딩, 기본적인 SQL 처리 담당                                        |
| `NamedParameterJdbcTemplate` | 이름 기반 파라미터 바인딩 지원 (가독성, 유지보수성 향상)                                     |
| `SimpleJdbcInsert`           | INSERT SQL 자동 생성, 편의 메서드 제공                                           |
| `RowMapper`                  | ResultSet → 객체 매핑                                                     |
| `BeanPropertyRowMapper`      | 컬럼명과 Java Bean 이름 자동 매핑 지원 (snake\_case → camelCase 변환 포함)            |
| `SqlParameterSource`         | 파라미터 전달용 객체 (MapSqlParameterSource, BeanPropertySqlParameterSource 등) |

---

## 🛠️ 사용 예시 및 기능별 정리

### 1. JdbcTemplate 기본 사용

```java
template.update(
  "INSERT INTO item (item_name, price, quantity) VALUES (?, ?, ?)",
  item.getItemName(), item.getPrice(), item.getQuantity()
);
```

### 2. 이름 지정 파라미터 바인딩 (NamedParameterJdbcTemplate)

```java
String sql = "UPDATE item SET item_name=:itemName WHERE id=:id";

SqlParameterSource param = new MapSqlParameterSource()
  .addValue("itemName", "상품명")
  .addValue("id", 1L);

template.update(sql, param);
```

### 3. BeanPropertySqlParameterSource 사용

```java
SqlParameterSource param = new BeanPropertySqlParameterSource(item);
// getter를 기반으로 파라미터 자동 매핑
```

### 4. RowMapper

```java
private RowMapper<Item> itemRowMapper() {
  return BeanPropertyRowMapper.newInstance(Item.class); // snake_case → camelCase 자동 변환
}
```

---

## 🚀 프로젝트 구성 변경 요약

| 단계 | 구현체                            | 특징                                 |
| -- | ------------------------------ | ---------------------------------- |
| V1 | `JdbcTemplateItemRepositoryV1` | 순서 기반 바인딩, 직접 쿼리 작성                |
| V2 | `JdbcTemplateItemRepositoryV2` | 이름 기반 바인딩, BeanProperty 활용         |
| V3 | `JdbcTemplateItemRepositoryV3` | `SimpleJdbcInsert` 사용으로 INSERT 간소화 |

---

## ⚠️ 주의할 점

* 순서 기반 바인딩은 쿼리 수정 시 **심각한 데이터 오류 발생** 가능
* 파라미터가 많아질수록 이름 기반 바인딩을 사용하여 안정성과 명확성 확보
* 동적 쿼리는 MyBatis나 QueryDSL 고려 필요

---

## 🔍 정리된 기능 요약

| 기능      | 메서드                                      | 설명                          |
| ------- | ---------------------------------------- | --------------------------- |
| 단건 조회   | `queryForObject()`                       | 숫자/문자/객체 단건 조회              |
| 다건 조회   | `query()`                                | 여러 Row를 리스트로 반환             |
| 변경      | `update()`                               | INSERT / UPDATE / DELETE 처리 |
| DDL 실행  | `execute()`                              | 테이블 생성 등                    |
| 저장 키 반환 | `SimpleJdbcInsert.executeAndReturnKey()` | 자동 생성 ID 반환                 |

---

## 💡 인사이트

* JdbcTemplate은 **JDBC 반복 작업을 대폭 줄여주며 실무에서 매우 유용**
* 그러나 SQL 직접 작성 및 동적 쿼리의 불편함 존재
* **MyBatis나 QueryDSL로의 전환 고려가 필요**
* 이름 기반 바인딩과 Bean 기반 파라미터 매핑은 유지보수성과 안정성 측면에서 필수

