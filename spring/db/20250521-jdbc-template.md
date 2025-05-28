# 데이터 접근 기술 - 스프링 JdbcTemplate

## ✅ 학습 주제

* JdbcTemplate의 등장 배경과 장점
* JdbcTemplate을 활용한 기본 CRUD 구현
* 이름 기반 바인딩을 위한 `NamedParameterJdbcTemplate`
* 자동 SQL 생성을 위한 `SimpleJdbcInsert`
* 구성 및 설정 방법
* 동적 쿼리의 한계와 대안

---

## 🧩 핵심 개념 요약

| 항목                         | 설명                                                         |
| -------------------------- | ---------------------------------------------------------- |
| JdbcTemplate               | JDBC 작업을 추상화하여 반복 코드를 제거해주는 스프링의 템플릿 클래스                   |
| 템플릿 콜백 패턴                  | 공통 로직(JDBC 반복 코드)은 프레임워크가 처리하고, 핵심 로직만 콜백으로 전달             |
| NamedParameterJdbcTemplate | `?` 대신 `:name` 형식으로 파라미터 바인딩하여 가독성과 안정성 향상                 |
| SimpleJdbcInsert           | INSERT SQL 없이도 테이블명, 컬럼만 설정하면 자동으로 삽입 쿼리 생성                |
| RowMapper                  | `ResultSet` → 도메인 객체 변환 담당 (람다 또는 `BeanPropertyRowMapper`) |

---

## 🛠️ 개발 단계별 흐름 정리

### 1. 설정

```groovy
// build.gradle
implementation 'org.springframework.boot:spring-boot-starter-jdbc'
runtimeOnly 'com.h2database:h2'
```

* JdbcTemplate은 `spring-jdbc`에 포함
* H2는 메모리 DB로 테스트용

---

### 2. 기본 구현 (`JdbcTemplateItemRepositoryV1`)

* `DataSource` → `JdbcTemplate` 생성
* `update()`로 데이터 삽입/수정/삭제
* `queryForObject()`로 단건 조회
* `query()`로 다건 조회
* `KeyHolder`로 PK 조회 처리

```java
KeyHolder keyHolder = new GeneratedKeyHolder();
template.update(con -> {
    PreparedStatement ps = con.prepareStatement(sql, new String[]{"id"});
    ...
    return ps;
}, keyHolder);
```

🔎 INSERT 후 **SELECT를 다시 실행하는 게 아니라**, JDBC의 `getGeneratedKeys()`를 통해 DB가 세션에서 기억하고 있는 생성 ID를 반환

---

### 3. 동적 쿼리 처리 (조건부 `findAll`)

* `WHERE`, `AND` 조건을 문자열로 수동 조합
* 파라미터는 `List<Object>`로 따로 관리

```java
if (itemName != null) sql += " WHERE item_name LIKE ?";
```

⚠ 유지보수 어려움 → MyBatis, QueryDSL 추천

---

### 4. 이름 기반 바인딩 (`JdbcTemplateItemRepositoryV2`)

```java
String sql = "UPDATE item SET item_name=:itemName WHERE id=:id";
SqlParameterSource param = new MapSqlParameterSource()
    .addValue("itemName", name).addValue("id", id);
```

* `NamedParameterJdbcTemplate` 사용
* `BeanPropertySqlParameterSource`, `MapSqlParameterSource` 활용
* **순서 실수 방지 + 명확한 SQL 표현 가능**

---

### 5. 자동 INSERT (`JdbcTemplateItemRepositoryV3`)

```java
SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(dataSource)
    .withTableName("item").usingGeneratedKeyColumns("id");

Number key = jdbcInsert.executeAndReturnKey(new BeanPropertySqlParameterSource(item));
```

* SQL 문 작성 없이 insert 가능
* 테이블 메타데이터를 분석하여 자동으로 SQL 생성
* KeyHolder 없이 키 반환 가능

---

## 📎 JdbcTemplate 기능 정리

| 기능                            | 설명                          |
| ----------------------------- | --------------------------- |
| `update()`                    | INSERT, UPDATE, DELETE      |
| `query()`, `queryForObject()` | 조회 (여러 건 / 단건)              |
| `RowMapper`                   | 결과 매핑 함수                    |
| `getGeneratedKeys()`          | INSERT 후 생성된 키 조회 (JDBC 표준) |
| `SimpleJdbcInsert`            | INSERT SQL 없이 삽입 처리         |
| `NamedParameterJdbcTemplate`  | 이름 기반 파라미터 처리               |

---

## 🔎 NamedParameterJdbcTemplate, BeanPropertySqlParameterSource, RowMapper 정리


## ✅ 1. NamedParameterJdbcTemplate

### 🔹 동작 원리

* 기존 `?` 방식이 아닌 `:name`으로 **이름 기반 파라미터 바인딩**을 제공
* SQL 가독성 향상 + 파라미터 순서 실수 방지
* 내부적으로 SQL을 `?`로 변환한 뒤 순서 매핑을 수행

### 🔹 예시

```java
String sql = "UPDATE item SET item_name = :itemName, price = :price WHERE id = :id";

Map<String, Object> param = Map.of(
    "itemName", "상품A",
    "price", 2000,
    "id", 1L
);

NamedParameterJdbcTemplate template = new NamedParameterJdbcTemplate(dataSource);
template.update(sql, param);
```

### ✅ 실무 포인트

* 파라미터 개수가 많거나, SQL 변경이 잦은 프로젝트에서는 **강력한 안정성 확보**에 효과적
* 쿼리 복잡도 높을수록 `NamedParameterJdbcTemplate`이 필수

---

## ✅ 2. BeanPropertySqlParameterSource

### 🔹 동작 원리

* 자바 객체의 getter 메서드를 분석해, **속성 이름 → SQL 파라미터 이름**으로 자동 매핑
* 예: `getItemName()` → `itemName`, `getPrice()` → `price`

### 🔹 예시

```java
public class Item {
    private String itemName;
    private int price;
    // getter/setter
}

Item item = new Item();
item.setItemName("상품B");
item.setPrice(3000);

String sql = "INSERT INTO item (item_name, price) VALUES (:itemName, :price)";
SqlParameterSource param = new BeanPropertySqlParameterSource(item);

NamedParameterJdbcTemplate template = new NamedParameterJdbcTemplate(dataSource);
template.update(sql, param);
```

### ✅ 실무 포인트

* DTO 객체나 도메인 객체를 직접 바인딩할 수 있어 **코드량 절감**
* 단, **객체 내에 SQL에 없는 필드**가 있다면 무시되며, 필요한 필드만 getter로 정의되어 있어야 함

---

## ✅ 3. RowMapper

### 🔹 동작 원리

* `ResultSet`을 자바 객체로 **매핑해주는 전략 객체**
* JDBC가 `rs.next()`로 데이터를 한 줄씩 읽을 때, 그 줄을 객체로 변환해주는 메서드

### 🔹 예시: 커스텀 RowMapper

```java
RowMapper<Item> rowMapper = (rs, rowNum) -> {
    Item item = new Item();
    item.setId(rs.getLong("id"));
    item.setItemName(rs.getString("item_name"));
    item.setPrice(rs.getInt("price"));
    return item;
};

String sql = "SELECT * FROM item WHERE id = :id";
Map<String, Object> param = Map.of("id", 1L);
Item result = template.queryForObject(sql, param, rowMapper);
```

### 🔹 예시: BeanPropertyRowMapper (자동 매핑)

```java
RowMapper<Item> rowMapper = BeanPropertyRowMapper.newInstance(Item.class);
```

* 컬럼명 `item_name` → 필드 `itemName` 자동 매핑
* 스네이크 케이스 ↔ 카멜 케이스 자동 변환

### ✅ 실무 포인트

* 단순 테이블 → 객체 매핑 시 `BeanPropertyRowMapper` 추천
* 성능이 민감하거나 컬럼명이 일치하지 않는 경우는 **커스텀 RowMapper 직접 구현**

---

## 🧠 정리 요약

| 도구                               | 역할                | 핵심 이점            |
| -------------------------------- | ----------------- | ---------------- |
| `NamedParameterJdbcTemplate`     | 이름 기반 파라미터 바인딩    | SQL 안정성, 가독성     |
| `BeanPropertySqlParameterSource` | 객체 필드 자동 추출       | DTO/VO 재활용 가능    |
| `RowMapper`                      | ResultSet → 객체 매핑 | 조회 결과 유연하게 처리 가능 |


---

## 💡 인사이트

* JdbcTemplate은 반복을 줄이고 가독성을 높여주는 강력한 도구지만, 복잡한 동적 쿼리는 비효율적
* 실무에서는 동적 쿼리를 위해 MyBatis, QueryDSL과 함께 쓰는 것이 일반적
* INSERT 후 ID 조회는 SELECT가 아닌 `getGeneratedKeys()`를 통한 DB 세션 기반 키 조회
* 설정을 수동으로 분리하고, 스캔 범위를 제한하여 테스트/운영 설정을 명확히 분리할 수 있음

