#  데이터 접근 기술 - 시작

## ✅ 스프링에서의 데이터 접근 기술이란?

* 애플리케이션이 **DB와 연결하여 데이터를 저장/조회/수정/삭제(CRUD)** 하기 위한 기술.
* 대표적으로 JDBC, JPA, 스프링 데이터 JPA 등이 있음.

---

## 📄 주요 개념 정리

### 1. **Repository**

* 데이터를 관리하는 인터페이스.
* 예시:

  ```java
  public interface ItemRepository {
      Item save(Item item);
      Optional<Item> findById(Long id);
      List<Item> findAll();
  }
  ```

### 2. **MemoryItemRepository (메모리 저장소 구현체)**

* 실제로 DB가 아닌 **HashMap** 등을 사용해서 데이터를 임시 저장하는 테스트용 구현체.
* 예시:

  ```java
  public class MemoryItemRepository implements ItemRepository {
      private static Map<Long, Item> store = new HashMap<>();
      ...
  }
  ```

### 3. **@SpringBootTest**

* 스프링 부트를 전체 실행해서 테스트 환경을 구성함.
* 실제 애플리케이션처럼 스프링 컨텍스트를 로딩하여 테스트 가능.

### 4. **@Autowired**

* 스프링 컨테이너에 등록된 Bean 중에서 자동으로 주입.

### 5. **@AfterEach**

* 각각의 테스트가 끝난 후 실행되는 메서드.
* 테스트 간 데이터 충돌을 막기 위해 메모리 저장소 초기화 용도로 사용함.

---

## 🧪 예시 코드 (테스트용)

```java
@SpringBootTest
class ItemRepositoryTest {
    @Autowired
    ItemRepository itemRepository;

    @AfterEach
    void afterEach() {
        if (itemRepository instanceof MemoryItemRepository) {
            ((MemoryItemRepository) itemRepository).clearStore();
        }
    }
}
```

---

## 🤔 궁금했던 점 & 해결

* **Q:** `itemRepository`는 어떤 구현체가 주입될까?
* **A:** `@SpringBootTest`는 전체 애플리케이션 컨텍스트를 로딩하므로, `ItemRepository`를 구현한 Bean 중 **단 하나**가 있으면 그것이 주입됨.
  여러 개라면 `@Primary`, `@Qualifier`, 또는 `@TestConfiguration`으로 명시해야 함.

---

## 💡 느낀 점

* 스프링이 객체를 자동으로 주입해주니까 편리하지만, 어떤 구현체가 선택되는지 정확히 이해해야 테스트가 꼬이지 않는다.
* 메모리 저장소로 먼저 구현하고, 나중에 JPA로 바꿔보면 흐름이 자연스럽게 익혀질 것 같다.

