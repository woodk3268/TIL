# 스프링 MVC 로그인 처리 2 (필터 & 인터셉터)

---

## ✅ 학습 주제

* 서블릿 필터와 스프링 인터셉터를 통한 인증 흐름 제어
* 공통 관심사(Cross-Cutting Concern)를 구조적으로 처리하는 방법

---

## 🧩 핵심 개념 요약

| 개념                    | 설명                                                |
| --------------------- | ------------------------------------------------- |
| 서블릿 필터(Filter)        | WAS 수준에서 작동하는 수문장. 요청 로그, 인증 필터 적용 가능             |
| 스프링 인터셉터(Interceptor) | 스프링 MVC에서 DispatcherServlet 이후에 작동. 컨트롤러 전후 제어 가능 |
| 공통 관심사                | 인증, 로깅 등 여러 컨트롤러에서 반복되는 처리. 필터나 인터셉터로 외부화함        |

---

## 🔍 필터 흐름 및 적용

### ✅ 필터 구조

```
HTTP 요청 → WAS → 필터 → 서블릿 → 컨트롤러
```

### 🔹 LogFilter

* UUID 생성 후 요청과 응답 로그 출력
* `FilterRegistrationBean`으로 등록 및 순서/URL 지정

### 🔹 LoginCheckFilter

* 화이트리스트 경로 외에는 세션 체크 수행
* 미인증 시 `/login?redirectURL=요청경로`로 리다이렉트
* 필터 체인 중단을 위해 `return` 처리 필수

### 🔹 단점 및 한계

* 요청 종료 후 흐름(afterCompletion) 제어 불가능
* URL 패턴 제어에 한계 있음 (`/*.ico`, `/css/*` 등만 가능)

---

## 🔍 인터셉터 흐름 및 적용

### ✅ 인터셉터 구조

```
HTTP 요청 → WAS → 필터 → 서블릿 → 인터셉터 → 컨트롤러
```

### 🔹 LogInterceptor

* `preHandle`: UUID 로그 식별자 설정
* `afterCompletion`: 예외 여부 포함 응답 로그 출력
* `handler` 정보 분석 가능 (핸들러 메서드 이름 등)

### 🔹 LoginCheckInterceptor

* `preHandle`에서 세션 검사
* 미인증 시 리다이렉트 후 `false` 반환으로 요청 중단

### 🔹 설정 방식

* `WebMvcConfigurer.addInterceptors()`로 URL 세밀 제어
  - 정적 리소스 요청을 제외할 수 있어 퍼포먼스 최적화 가능
  - REST API에 대해서만 적용할 수도 있고, 정규식 기반 경로 변수 매칭도 지원
* `addPathPatterns`, `excludePathPatterns`로 정적 리소스, 로그인, 홈 등 제외 가능

### 🔹 장점

* 예외 발생 시에도 afterCompletion 호출 보장
* 컨트롤러 정보(`HandlerMethod`) 접근 가능
* 서블릿 필터보다 스프링 환경에 유연하게 대응

## 🔍 필터 VS 인터셉터

### ✅ 1. 요청 정보 처리 수준의 차이
| 항목         | 서블릿 필터                                     | 스프링 인터셉터                                                                                     |
| ---------- | ------------------------------------------ | -------------------------------------------------------------------------------------------- |
| 요청 처리 위치   | DispatcherServlet **이전**                   | DispatcherServlet **이후**, 컨트롤러 **직전/직후**                                                     |
| 객체 정보 접근   | `ServletRequest`, `ServletResponse`만 접근 가능 | `HttpServletRequest`, `HttpServletResponse` + **HandlerMethod, ModelAndView 등** MVC 정보 접근 가능 |
| 컨트롤러 정보    | 알 수 없음                                     | `HandlerMethod`를 통해 어떤 컨트롤러 메서드가 호출되는지 파악 가능                                                 |
| 세션 파라미터 주입 | 수동 조회 필요 (`request.getSession()`)          | `ArgumentResolver` 등과 조합해 자동 주입 처리 가능                                                        |

### ✅ 2. URL 제어 및 설정 유연성
| 항목        | 서블릿 필터                                 | 스프링 인터셉터                                                                                |
| --------- | -------------------------------------- | --------------------------------------------------------------------------------------- |
| URL 패턴 지정 | 서블릿 수준 (`/*`, `*.do` 등 제한적)            | `addPathPatterns`, `excludePathPatterns`로 **세밀하게 제어** (`/api/**`, `/members/{id}` 등 가능) |
| 리소스 제외 처리 | 정적 파일 경로 하드코딩                          | `.excludePathPatterns("/css/**", "/*.ico")`처럼 **명시적으로 지정 가능**                           |
| 필터 순서 제어  | `FilterRegistrationBean.setOrder()` 필요 | `.order()`로 명시적인 인터셉터 순서 지정 가능                                                          |

### ✅ 3. 예외 처리 및 로깅 지원
- 서블릿 필터는 예외가 발생해도 로그 후 처리 흐름을 제어하기 어렵고, 응답도 직접 조작해야 함
- 스프링 인터셉터는 afterCompletion()에서 예외 객체(ex)를 전달받아 공통 예외 처리 가능
- 즉, MVC 흐름과 연동된 에러 로깅/트랜잭션 정리 등 비즈니스 친화적 처리 가능

### ✅ 4. 스프링 기능과의 통합성

| 기능                  | 서블릿 필터   | 스프링 인터셉터                                           |
| ------------------- | -------- | -------------------------------------------------- |
| `@ControllerAdvice` | 연동 안 됨   | MVC 예외 흐름에 통합됨                                     |
| AOP 연계              | 어려움      | `preHandle`, `postHandle` 단계에서 AOP처럼 작동 가능         |
| 스프링 Security와 연계    | 별도 구현 필요 | Security filter chain보다 후처리로 사용 가능 (세분화된 시점 활용 가능) |

---

## 🧠 인사이트

* 필터는 WAS에서 DispatcherServlet 이전에 실행되므로 **낮은 단계의 공통 처리에 적합** (ex. XSS 방어, IP 차단)
* 인터셉터는 DispatcherServlet 이후, HandlerMapping → Controller 전후에서 동작하므로 **컨트롤러 중심 인증/인가 로직에 적합**


