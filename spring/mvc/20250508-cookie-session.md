# 스프링 MVC 로그인 처리 1 (쿠키 & 세션)

---

## ✅ 학습 주제

* 로그인 요구사항 분석 및 흐름 구현
* 쿠키 기반 로그인 상태 유지 방법
* 세션 직접 구현 및 서블릿 `HttpSession` 활용

---

## 🧩 핵심 구성요소 요약

| 구성 요소               | 설명                                                  |
| ------------------- | --------------------------------------------------- |
| 쿠키 (`Cookie`)       | 클라이언트에 저장되어 서버와의 상태 유지를 도와주는 데이터 조각                 |
| 세션 (`Session`)      | 서버 측에서 사용자의 상태 정보를 관리하는 방법. 식별자(SessionId)는 쿠키로 전달됨 |
| SessionManager      | 직접 만든 세션 관리 도구. UUID 기반 식별자 생성, 세션 저장소, 만료 기능 포함    |
| `HttpSession`       | 서블릿에서 제공하는 표준 세션 기능. `JSESSIONID` 쿠키 기반             |
| `@SessionAttribute` | 세션 데이터를 편리하게 조회할 수 있는 Spring MVC의 어노테이션             |

---

## 🛠️ 단계별 흐름 요약

### 1. 로그인 요구사항

* 로그인 전: 회원가입 / 로그인 버튼만 표시
* 로그인 후: 이름 환영 메시지 + 상품 관리 / 로그아웃 버튼 표시
* 비로그인 시 상품 접근 불가 → 로그인 페이지 리다이렉트

### 2. 쿠키 기반 로그인 상태 유지

* 로그인 성공 시 `memberId` 쿠키 저장
* 홈 컨트롤러에서 `@CookieValue`로 사용자 정보 조회 후 로그인 여부 판단
* 로그아웃 시 `MaxAge = 0`으로 쿠키 즉시 만료

📛 **문제점**

* 쿠키는 클라이언트에서 조작 가능
* 보안 정보 노출 우려 (예: 다른 사용자 ID로 변경 가능)

🔐 **대안**

* 예측 불가능한 토큰 발급 → 서버 저장소에서 사용자와 토큰 매핑 → 쿠키에는 토큰만 저장

### 3. 세션 직접 구현

* `SessionManager`에서 UUID 기반 식별자 생성 → 쿠키에 저장
* 내부 Map에 사용자 데이터 저장 (`ConcurrentHashMap` 사용)
* 테스트는 `MockHttpServletRequest/Response` 활용

### 4. 직접 구현한 세션 적용

* 로그인 시 `SessionManager.createSession()` 호출하여 세션 생성
* 홈 컨트롤러에서 세션 조회 후 로그인 여부 판단
* 로그아웃 시 `SessionManager.expire()` 호출

### 5. 서블릿 HTTP 세션 활용

* `request.getSession(true)`로 세션 생성 및 조회
* 사용자 정보는 `session.setAttribute("loginMember", member)` 방식으로 저장
* 로그아웃 시 `session.invalidate()` 호출로 세션 제거

### 6. `@SessionAttribute` 활용

* 세션 접근시 반복되는 코드 제거 가능

```
@SessionAttribute(name = "loginMember", required = false) Member loginMember
```

---

## 📂 세션 타임아웃 설정 요약

| 설정 위치                    | 설명                                               |
| ------------------------ | ------------------------------------------------ |
| `application.properties` | `server.servlet.session.timeout=60` (초 단위) 설정 가능 |
| 개별 세션 설정                 | `session.setMaxInactiveInterval(1800)` → 30분     |

⏳ 최근 접근 시간 기준으로 세션 연장됨 (LastAccessedTime 기반)

---

## 💡 인사이트

* 서버에서 중요한 정보를 관리하고, 클라이언트에는 식별자만 보낸다는 것이 핵심
* 직접 만든 세션은 개념을 익히기 위한 용도로 적합, 실제로는 `HttpSession` 활용이 안정적이고 효율적
* 세션 사용 시 사용자 수 \* 보관 데이터 용량 → 메모리 부담 가능성 있으므로 최소한의 데이터만 저장해야 함


