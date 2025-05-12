# 스프링 MVC 예외 처리와 오류 페이지

---

## ✅ 학습 주제

* 서블릿과 스프링 기반 예외 처리 흐름의 차이 이해
* 오류 발생 시 클라이언트 응답 방식과 내부 오류 페이지 흐름
* `Filter`, `Interceptor`, `DispatcherType`, `BasicErrorController`의 활용

---

## 🧩 핵심 개념 요약

| 개념                         | 설명                                           |
| -------------------------- | -------------------------------------------- |
| `Exception`, `sendError()` | 예외 발생 시 WAS에 전달되는 방식, HTTP 상태 코드 기반 오류 응답    |
| 오류 페이지 등록                  | 상태 코드 또는 예외 타입에 따라 사용자 정의 오류 페이지 연결          |
| `DispatcherType`           | 요청 분류 타입: `REQUEST`, `ERROR`, `FORWARD` 등    |
| `Filter` 오류 분기 처리          | `setDispatcherTypes()`로 오류 페이지 호출 시 필터 제외 가능 |
| `Interceptor` 오류 분기 처리     | `excludePathPatterns()`로 오류 페이지 URI 제외       |
| `BasicErrorController`     | 스프링 부트에서 `/error`로 오류 요청을 받는 기본 오류 컨트롤러      |

---

## 🛠️ 처리 흐름 요약

### 1. 기본 서블릿 예외 처리 흐름

* 컨트롤러 예외 발생 → WAS까지 예외 전달 → 기본 오류 페이지 표시 (`500`, `404`)
* `response.sendError(statusCode, message)` 호출 시에도 동일하게 처리됨

### 2. 사용자 정의 오류 페이지 등록

* `WebServerCustomizer`에서 `ErrorPage(HttpStatus, URI)` 등록
* 예외 발생 시 지정된 경로로 내부 재요청됨
* 오류 정보를 `request.attribute`에 담아 컨트롤러에서 활용 가능

### 3. 오류 요청과 일반 요청의 구분

* `DispatcherType`으로 클라이언트 직접 요청 vs WAS 내부 오류 재요청 구분 가능
* `Filter`에서 `DispatcherType.REQUEST`만 허용 시 오류 재요청 필터 제외
* `Interceptor`는 무조건 호출되므로 `/error-page/**` 경로 제외 필수

### 4. 스프링 부트 기본 오류 처리

* `BasicErrorController`가 `/error` 요청 처리
* 뷰 우선순위: `templates/error/500.html` > `static/error/500.html` > `error.html`
* 개발자는 오류 페이지 템플릿만 정의하면 됨

### 5. 오류 페이지에서 표시 가능한 정보

* `timestamp`, `status`, `error`, `message`, `exception`, `trace`, `path`, `errors`
* `application.properties` 설정으로 노출 범위 제어 가능 (예: `include-message=on_param`)

---

### 💡 인사이트

* WAS는 오류 발생 시 예외를 다시 서블릿 흐름으로 전달해 `/error`나 `/error-page/**`를 재요청함
* 예외 상황에서도 로그 필터와 인터셉터가 작동할 수 있으므로, 반드시 중복 호출 방지 설정 필요
