# 학습 주제
- Spring MVC에서의 서버 사이드 검증 처리 방법
- BindingResult, Validator, 메시지 국제화 적용을 통한 오류 메시지 처리

# 핵심 구성요소 요약

| 구성 요소 | 설명 |
|-----------|------|
| `BindingResult` | 컨트롤러에서 검증 오류 정보를 담는 객체. 바인딩 오류 발생 시 컨트롤러 진입 가능 |
| `FieldError` / `ObjectError` | 필드/글로벌 오류 정보 객체로 오류 메시지, 필드명, 값 등을 포함 |
| `rejectValue()`, `reject()` | 오류 메시지를 자동 생성하고 메시지 코드 기반으로 메시지 해결 |
| `Validator` | 검증 로직을 컨트롤러 외부 클래스로 분리하여 재사용성과 유지보수성 향상 |
| `MessageCodesResolver` | 오류 메시지 코드 순위를 생성 (구체적 → 범용적 메시지 검색 순서 구성) |

---

# 단계별 검증 적용 흐름

## 1. 기본 검증 요구사항
- `itemName` 필수
- `price`: 1,000 ~ 1,000,000
- `quantity`: 최대 9,999
- `price * quantity`: 10,000 이상

## 2. 컨트롤러 내 직접 검증 처리 (V1)
- `Map<String, String>` 으로 오류 메시지 수동 관리
- 뷰에서 `errors['필드명']`으로 접근
- **단점**: 코드 중복

## 3. `BindingResult` 활용 (V2~V4)
- `FieldError`, `ObjectError` 사용 → 사용자 입력 값 유지
- 메시지 파일 기반 오류 메시지 관리
- `rejectValue()`, `reject()`로 간결한 오류 처리 가능

**addError()와 rejectValue()의 차이점**

| 항목                         | `addError()`                                      | `rejectValue()`                                   |
|----------------------------|--------------------------------------------------|--------------------------------------------------|
| 직접 생성 여부              | `FieldError` / `ObjectError` 직접 생성              | 자동 생성                                         |
| 메시지 코드 자동 생성       | ❌ 수동 입력 필요                                   | ✅ `MessageCodesResolver` 기반 자동 생성           |
| 사용자 입력값 유지 (`rejectedValue`) | ✅ 명시적으로 설정                                | ✅ 내부에서 자동 저장                              |
| 코드 간결성                 | ❌ 길고 복잡                                       | ✅ 간결                                           |
| 사용 용도                   | 커스터마이징, 메시지 로직 세밀 제어                   | 일반적인 필드 검증                                 |

1) addError()
  - 직접 FieldError 또는 ObjectError 객체를 생성해서 BindingResult에 넣는 방식
  - 오류 메시지 코드 자동 생성 없음
  - 정교하게 오류 객체를 구성하고 싶을 때 사용
  - ```
    bindingResult.addError(new FieldError("item", "price", 500, false, null, null, "가격은 1000 이상이어야 합니다."));
    ```
  - ✅ 장점
    - rejectedValue, bindingFailure, codes, arguments, defaultMessage 등 모든 필드를 세밀하게 설정 가능
    - 메시지 처리 로직을 커스터마이징할 수 있음
   
  - ❌ 단점
    - 코드가 길고 복잡함
    - 메시지 코드를 직접 세팅해야 함
      
2) rejectValue()
  - Spring이 자동으로 FieldError를 생성해서 BindingResult에 넣는 간편 메서드
  - MessageCodesResolver가 자동으로 메시지 코드 순서를 생성
  - 주로 일반적인 폼 검증에서 사용
  - ```
    bindingResult.rejectValue("price", "range", new Object[]{1000, 1000000}, null);
    ```
  - ✅ 장점
  - 코드가 간결함
  - 메시지 코드 체계(range.item.price, range.price, ... range)를 Spring이 자동 생성
  - 국제화 및 메시지 재사용에 유리
  
  - ❌ 단점
  - 복잡한 상황(예: 여러 개의 메시지 코드, 사용자 정의 메시지 순서 등)에선 유연성이 떨어짐

3) 추천 기준
- 단순한 검증 → rejectValue() / reject() 사용
- 복잡한 메시지, 커스텀 로직, 특별한 rejectedValue 처리 → addError() 사용

## 4. 메시지 코드 전략 (V3~V4)
- `range.item.price` → `range.price` → `range.java.lang.Integer` → `range` 순
- `errors.properties`로 메시지 분리 관리
- 국제화 대응 가능 (e.g., `errors_ko.properties`, `errors_en.properties`)

## 5. 검증 로직 분리 (V5~V6)
- `ItemValidator implements Validator`
- `@InitBinder` + `@Validated`로 자동 적용
- 글로벌 설정은 `WebMvcConfigurer`에서 `getValidator()` 오버라이드

---

# 💡 인사이트
- 오류 메시지를 코드화하면 다국어 및 재사용성이 극대화됨
- `BindingResult`를 사용하면 타입 오류 시에도 사용자 입력값을 보존할 수 있어 UX 향상
- Validator는 관심사 분리에 강력한 수단으로 활용 가능


