# 데이터 접근 기술 - 스프링 데이터 JPA

---

## ✅ 학습 주제

* 스프링 데이터 JPA의 등장 배경과 주요 기능
* `JpaRepository` 기반 CRUD 및 쿼리 메서드 사용
* JPQL 직접 작성 vs 메서드 이름 자동 파싱 방식 비교
* 기존 `ItemRepository`와 스프링 데이터 JPA 연동 방식

---

## 🧩 핵심 개념 요약

| 항목              | 설명                                            |
| --------------- | --------------------------------------------- |
| `JpaRepository` | CRUD 등 공통 기능을 제공하는 최상위 인터페이스                  |
| 쿼리 메서드          | 메서드 이름으로 JPQL 자동 생성 (예: `findByItemNameLike`) |
| `@Query`        | JPQL을 직접 작성 가능                                |
| `@Param`        | `@Query` 내 파라미터 바인딩                           |
| 동적 쿼리           | 스프링 데이터 JPA에서는 약함 → **QueryDSL 사용 권장**        |
| 예외 변환           | 프록시 내부에서 자동 처리되어 `@Repository` 없이도 지원됨        |

---

## 🛠️ 스프링 데이터 JPA 적용 요약

### 1. Repository 인터페이스 정의

```java
public interface SpringDataJpaItemRepository extends JpaRepository<Item, Long> {
    List<Item> findByItemNameLike(String itemName);
    List<Item> findByPriceLessThanEqual(Integer price);
    List<Item> findByItemNameLikeAndPriceLessThanEqual(String itemName, Integer price);

    @Query("select i from Item i where i.itemName like :itemName and i.price <= :price")
    List<Item> findItems(@Param("itemName") String itemName, @Param("price") Integer price);
}
```

---

### 2. 스프링 데이터 JPA 기반 구현체 생성 (어댑터)

```java
@Repository
@Transactional
@RequiredArgsConstructor
public class JpaItemRepositoryV2 implements ItemRepository {

    private final SpringDataJpaItemRepository repository;

    public Item save(Item item) { return repository.save(item); }

    public void update(Long itemId, ItemUpdateDto dto) {
        Item item = repository.findById(itemId).orElseThrow();
        item.setItemName(dto.getItemName());
        item.setPrice(dto.getPrice());
        item.setQuantity(dto.getQuantity());
    }

    public Optional<Item> findById(Long id) { return repository.findById(id); }

    public List<Item> findAll(ItemSearchCond cond) {
        String itemName = cond.getItemName();
        Integer maxPrice = cond.getMaxPrice();
        if (hasText(itemName) && maxPrice != null) return repository.findItems("%" + itemName + "%", maxPrice);
        else if (hasText(itemName)) return repository.findByItemNameLike("%" + itemName + "%");
        else if (maxPrice != null) return repository.findByPriceLessThanEqual(maxPrice);
        else return repository.findAll();
    }
}
```

---

### 3. 설정 클래스

```java
@Configuration
@RequiredArgsConstructor
public class SpringDataJpaConfig {

    private final SpringDataJpaItemRepository springDataJpaItemRepository;

    @Bean
    public ItemService itemService() {
        return new ItemServiceV1(itemRepository());
    }

    @Bean
    public ItemRepository itemRepository() {
        return new JpaItemRepositoryV2(springDataJpaItemRepository);
    }
}
```

---

## 📚 주요 기능 비교

| 기능          | 설명                                        |
| ----------- | ----------------------------------------- |
| **기본 CRUD** | `JpaRepository`가 모두 제공 (save, findById 등) |
| **쿼리 메서드**  | 메서드명으로 자동 JPQL 생성 (단순 조건일 때 유용)           |
| **@Query**  | 복잡한 쿼리 직접 작성 가능                           |
| **동적 쿼리**   | 조건 분기 로직 수동 처리 → QueryDSL로 보완             |
| **예외 처리**   | 자동 변환되어 `DataAccessException`으로 처리됨       |

---

## ⚠️ 한계점 및 개선 방향

| 한계            | 보완 방법                       |
| ------------- | --------------------------- |
| 동적 조건 쿼리 미흡   | QueryDSL 도입                 |
| 메서드명이 길어지는 문제 | `@Query` 또는 사용자 정의 리포지토리 사용 |
| 복잡한 조인, 서브쿼리  | JPQL 또는 QueryDSL 활용         |

---

## 💡 인사이트

* **스프링 데이터 JPA는 JPA의 생산성을 극대화**하는 도구
* 반복되는 CRUD/조회 코드를 줄이고, 구현체 없이 인터페이스만으로 개발 가능
* 실무에서는 **단순 조회는 쿼리 메서드**, **복잡 조회는 `@Query` or QueryDSL**을 병행하는 방식이 일반적
* `JpaItemRepositoryV2` 어댑터 구조 덕분에 서비스 계층 코드를 수정하지 않고 기술 교체 가능


