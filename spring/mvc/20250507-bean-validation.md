# 스프링 MVC Validation(검증2 - Bean Validation)

## ✅ 학습 주제
- Bean Validation의 개념 및 스프링 통합 방법
- 애노테이션 기반 검증 적용 및 커스터마이징
- groups와 Form 객체 분리 방식에 따른 실무 대응 전략

---

## 🧩 핵심 구성요소 요약

| 구성 요소 | 설명 |
|-----------|------|
| `@NotBlank`, `@NotNull`, `@Range`, `@Max` | 필드 검증을 위한 표준 및 구현체 애노테이션 |
| `@Validated`, `@Valid` | Bean Validation을 트리거하는 어노테이션 (`@Validated`는 그룹 기능 포함) |
| `BindingResult` | 애노테이션 검증 오류도 담겨서 처리 가능 |
| `MessageCodesResolver` | `NotBlank.item.itemName` → `NotBlank.itemName` → ... 순으로 메시지 탐색 |
| Form 객체 분리 | `ItemSaveForm`, `ItemUpdateForm`으로 등록/수정 조건 분리 |
| 그룹 검증 (`groups`) | 저장/수정 시 요구사항이 다를 경우 그룹을 통해 검증 조건 분기 |
| 글로벌 오류 처리 | `bindingResult.reject(...)` 사용해 객체 전체의 로직 오류 처리 |
| @RequestBody와 검증 | `@Validated`가 JSON에도 적용되며, 실패 시 400 응답 발생 |

---

## 🛠️ 단계별 검증 적용 흐름

### 1. 의존성 설정
- `spring-boot-starter-validation` 추가 → Hibernate Validator 자동 등록

### 2. 애노테이션 기반 검증 적용
- `Item` 클래스에 검증 애노테이션 적용 (`@NotBlank`, `@Range` 등)

### 3. 스프링 컨트롤러에 적용
```
@PostMapping("/add")
public String addItem(@Validated @ModelAttribute Item item, BindingResult bindingResult) {
    ...
}
```

### 4. 글로벌 오류 처리
- 예: 가격 × 수량 < 10,000인 경우 → `bindingResult.reject("totalPriceMin", ...)` 사용

### 5. 수정/등록 분리 문제
- `Item` 하나만 쓰면 등록과 수정 요구사항이 충돌함 (`@NotNull id`, `@Max quantity`)

### 6. 해결 방법
- ✅ 방법 1: `groups` 기능 사용 → `@Validated(SaveCheck.class)`
- ✅ 방법 2: Form 객체 분리 → `ItemSaveForm`, `ItemUpdateForm`

### 7. API 요청 검증 (`@RequestBody`)
- JSON을 객체로 바인딩 후 `@Validated` 적용
- Jackson 바인딩 실패 시 400 오류 발생
- Validator 실패 시 오류 객체 JSON으로 반환 가능

---
## ※ 왜 @RequestBody는 컨트롤러 진입이 안 되나?
-@RequestBody는 JSON → 자바 객체 변환을 위해 HttpMessageConverter를 사용해 요청 본문을 파싱하는데, 이 단계에서 타입이 맞지 않으면 예외를 바로 던져버린다.

## ※ 반면, @ModelAttribute는 왜 오류가 나도 들어올 수 있나?
- 폼 요청에서 입력값은 문자열이다.
- 스프링은 이 문자열을 ConversionService로 변환 시도한다.
- 실패하면 해당 필드만 FieldError로 처리해서 BindingResult에 담는다.
- 나머지 필드는 바인딩 성공 → 컨트롤러 진입 가능

---

## 💡 인사이트
- 검증 애노테이션만으로 간단한 규칙을 명확하게 표현할 수 있어 코드 가독성과 유지보수성이 높아진다.
- Form 객체 분리를 통해 검증 충돌 방지 및 역할 명확화 가능
- JSON 요청의 경우 바인딩 자체가 실패하면 컨트롤러 진입조차 안 되므로 사전 포맷 검증이 필요
