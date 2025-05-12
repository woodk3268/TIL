# API 예외 처리

---

## ✅ 학습 주제

* 스프링 MVC 환경에서 API 예외를 JSON 형태로 깔끔하게 처리하는 다양한 방식 학습
* 서블릿 오류 페이지부터, ExceptionResolver, @ExceptionHandler, @ControllerAdvice까지 예외 처리 전략별 비교

---

## 🧩 예외 처리 전략 요약

| 방식                                           | 설명                                              |
| -------------------------------------------- | ----------------------------------------------- |
| `BasicErrorController`                       | 스프링 부트 기본 오류 처리 컨트롤러. HTML 및 JSON 응답 자동 처리      |
| `ErrorPage` + produces JSON                  | 오류 페이지 URL에 JSON 응답을 추가 설정 (produces 조건)        |
| `HandlerExceptionResolver`                   | 예외를 가로채어 직접 상태 코드 및 응답 제어 가능, 서블릿까지 전달 없이 처리 가능 |
| `@ResponseStatus`, `ResponseStatusException` | 예외에 상태 코드를 매핑하여 자동 응답 처리 가능                     |
| `@ExceptionHandler`                          | 컨트롤러 내 개별 예외를 잡아 JSON 포맷 반환 가능                  |
| `@ControllerAdvice`                          | 예외 핸들러를 전역 공통으로 추출해 재사용성과 유지보수성 향상              |

---

## 🛠️ 단계별 예외 처리 흐름

### 1. 오류 페이지 + JSON

* `/error-page/500` 등의 오류 페이지 컨트롤러에 `produces = application/json` 설정
* `RequestDispatcher.ERROR_EXCEPTION` 등을 이용해 예외 내용 구성 후 JSON 반환

### 2. BasicErrorController 활용

* `/error` 요청 자동 처리
* Accept header에 따라 JSON or HTML 자동 선택
* `server.error.include-message`, `trace` 설정 가능
* 모든 예외를 동일한 형식으로 반환함 → API마다 다른 응답 구조 불가

### 3. HandlerExceptionResolver

* 예외가 발생하더라도 WAS까지 전달하지 않고 스프링에서 자체 처리
* 직접 예외를 가로채 `sendError`, `response.getWriter().write(...)` 등 응답 수동 제어
* `ModelAndView` 반환 값에 따라 흐름 분기 가능
* `extendHandlerExceptionResolvers()`에서 Resolver 등록
* 구현이 번거롭고, HttpServletResponse 직접 다뤄야 함

### 4. ResponseStatus 계열

* `@ResponseStatus`: 예외 클래스에 부여 시 해당 상태 코드 자동 적용
* `ResponseStatusException`: 런타임에 상태코드 + 메시지 던짐 가능

### 5. 스프링 내장 ExceptionResolver

* `ExceptionHandlerExceptionResolver`: `@ExceptionHandler` 처리 (우선순위 가장 높음)
* `ResponseStatusExceptionResolver`: 애노테이션 기반 상태 응답 처리
* `DefaultHandlerExceptionResolver`: 타입 바인딩 오류 등 내부 예외 → 400 응답으로 전환

### 6. @ExceptionHandler 활용

* 컨트롤러 내 메서드에 `@ExceptionHandler(Exception.class)` 등으로 예외 전용 메서드 구현
* `ResponseEntity` 반환 시 상태 코드 직접 설정 가능
* 컨트롤러 단위의 예외 맞춤 처리 가능

### 7. @ControllerAdvice

* 전역 예외 핸들러 설정 가능 (`@RestControllerAdvice` → 자동 JSON 변환)
* 특정 패키지, 타입, 애노테이션 기준으로 대상 지정 가능
* 개별 예외별 공통 처리 클래스 분리 가능

---

## 💡 인사이트

* `BasicErrorController`는 간단한 웹 오류 처리에는 충분하지만, 세밀한 API 예외 대응에는 부족함
* `ExceptionResolver`는 서블릿까지 예외 전달을 차단하고 흐름을 제어할 수 있는 장점이 있음
* `@ExceptionHandler + @ControllerAdvice` 조합은 가장 실무적이며 선언적이고 유지보수성이 높음

