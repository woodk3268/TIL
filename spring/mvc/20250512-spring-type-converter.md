# 스프링 타입 컨버터

---
### ✅ 학습 주제

* 스프링 MVC의 요청/응답 처리에서 발생하는 타입 변환 처리 방식 이해
* `Converter`, `Formatter`, `ConversionService`의 개념과 적용 방법 학습

---

### 🧩 핵심 개념 요약

| 개념                                 | 설명                                          |
| ---------------------------------- | ------------------------------------------- |
| `Converter<S, T>`                  | 일반적인 타입 변환 (문자 → 숫자, 객체 → 문자 등). 전 방향 지원 가능 |
| `Formatter<T>`                     | 주로 사용자 입력/출력에 적합한 문자 변환기. Locale 지원 포함      |
| `ConversionService`                | 여러 컨버터/포맷터를 등록하고 관리하는 타입 변환 도우미             |
| `@NumberFormat`, `@DateTimeFormat` | 애노테이션 기반 포맷 지정 기능                           |

---
### 🛠️ 사용 흐름 및 적용 방식

### 1. 기본 컨버터 구현

* `Converter<String, Integer>` 또는 `Converter<IpPort, String>` 등 구현
* `convert()` 메서드에서 명시적으로 변환 처리
* 예: `StringToIntegerConverter`, `IpPortToStringConverter`
#### 2. ConversionService 등록 및 사용

* `DefaultConversionService` 또는 `DefaultFormattingConversionService` 사용
* `addConverter()` 또는 `addFormatter()`로 등록
* `convert(source, targetType)` 방식으로 변환 호출

### 3. WebMvcConfigurer로 스프링에 적용

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new StringToIpPortConverter());
        registry.addFormatter(new MyNumberFormatter());
    }
}
```

### 4. 요청 파라미터 변환 (`@RequestParam`, `@ModelAttribute`, `@PathVariable`)

* `?data=10` 쿼리 → Integer로 자동 변환
* `?ipPort=127.0.0.1:8080` → IpPort 객체로 자동 바인딩

### 5. 뷰 렌더링 적용

* `${{number}}`, `${{ipPort}}` → 뷰에서 컨버터 적용 결과 출력

### 6. 폼 처리 적용

* `th:field="*{ipPort}"` → 입력 폼에서도 자동 타입 변환 지원

---

## 🎨 포맷터 사용 및 애노테이션 기반 설정

| 기능                       | 설명                            |
| ------------------------ | ----------------------------- |
| `MyNumberFormatter`      | 숫자 ↔ 쉼표 포함 문자열 포맷팅 (`1,000`)  |
| `@NumberFormat(pattern)` | 숫자 필드에 특정 포맷 지정               |
| `@DateTimeFormat`        | `LocalDateTime` 등 날짜 필드 포맷 지정 |

### 예시

```java
@NumberFormat(pattern = "###,###")
private Integer number;

@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
private LocalDateTime localDateTime;
```

---

## 🔄 ConversionService vs HttpMessageConverter

| 구분       | ConversionService                           | HttpMessageConverter (예: Jackson)                  |
| -------- | ------------------------------------------- | -------------------------------------------------- |
| 사용 위치    | @RequestParam, @ModelAttribute, 뷰 템플릿 렌더링 등 | @RequestBody, @ResponseBody (HTTP 본문 처리)           |
| 작동 시점    | URL 파라미터, 폼 필드, 경로 변수 등                     | JSON/XML 요청 본문 또는 응답 본문                            |
| 포맷 설정 방식 | @DateTimeFormat, @NumberFormat              | @JsonFormat, Jackson 설정                            |
| 예외       | JSON → 객체 변환 시에는 ConversionService 작동 ❌     | View 렌더링, 쿼리 파라미터 변환 시에는 HttpMessageConverter 작동 ❌ |

### 오해 방지 포인트

* `@DateTimeFormat`은 JSON 요청 바디에서는 작동하지 않음 → Jackson 사용
* JSON → 객체 변환 시 숫자/날짜 포맷 변경하려면 `@JsonFormat` 사용해야 함
---

## 💡 인사이트

* 스프링은 내부적으로 `ConversionService`를 통해 거의 모든 요청 데이터 타입(예: 문자열 → 숫자, 문자열 → 날짜, 문자열 → 사용자 정의 객체 등)을 자동으로 변환함
* 사용자 정의 컨버터를 등록하면 기존 컨버터보다 우선순위를 가지므로 커스터마이징 가능
* 포맷터는 로케일 지원과 사용자 입출력 포맷 처리에 강력하며, 뷰 템플릿과의 결합도 뛰어남
* HTTP 바디(JSON)는 `HttpMessageConverter`에서 처리되므로, 타입 컨버터와는 별개 흐름임

---





