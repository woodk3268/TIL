# 스프링 파일 업로드 처리

---

## ✅ 학습 주제

* HTML의 multipart 요청 방식 이해
* 서블릿과 스프링 기반 파일 업로드 처리 흐름 비교
* 스프링의 MultipartFile 활용법과 실제 파일 저장 구현

---

## 🧩 핵심 개념 요약

| 개념/도구                               | 설명                                                |
| ----------------------------------- | ------------------------------------------------- |
| `application/x-www-form-urlencoded` | 일반 텍스트 데이터 전송 (파일 업로드 불가)                         |
| `multipart/form-data`               | 텍스트 + 바이너리 데이터를 함께 전송할 수 있는 폼 방식                  |
| `Part` (Servlet)                    | HTTP multipart 요청에서 각 파트를 표현하는 표준 인터페이스           |
| `MultipartFile` (Spring)            | 스프링이 제공하는 업로드 파일 추상화, 간단하고 강력함                    |
| `MultipartResolver`                 | 스프링에서 multipart 요청을 분석하고 MultipartFile로 변환하는 컴포넌트 |

---

## 🛠️ 처리 흐름 및 예제 요약

### 1. HTML 폼 전송 방식

* 파일 업로드에는 `enctype="multipart/form-data"` 필수
* `application/x-www-form-urlencoded`는 바이너리 전송 불가

### 2. 서블릿 기반 파일 업로드 (V1/V2)

* `HttpServletRequest.getParts()`로 파트 접근
* `Part.getSubmittedFileName()`, `getInputStream()`, `write(...)`로 저장
* 설정: `spring.servlet.multipart.enabled=true`, `file.dir`로 저장 위치 지정

### 3. 스프링 MultipartFile 사용

```java
@PostMapping("/upload")
public String upload(@RequestParam MultipartFile file) {
    file.transferTo(new File("저장경로"));
}
```

* `MultipartFile.getOriginalFilename()` : 업로드 파일명
* `transferTo(...)` : 서버 저장

---


## ✅ Q1. MultipartResolver란?

> 스프링에서 **`multipart/form-data` 형식의 HTTP 요청을 파싱**해서
> 개발자가 쉽게 `MultipartFile` 형태로 사용할 수 있게 도와주는 **전처리기(프록시 컴포넌트)**

---

## 📦 언제 작동하는가?

### 🔹 요청 조건

* 요청의 `Content-Type`이 `multipart/form-data`일 때만 작동

### 🔹 작동 시점

1. 클라이언트가 파일 업로드 요청
2. **DispatcherServlet이 요청을 받기 전**, 스프링은 등록된 `MultipartResolver`로 요청을 검사
3. multipart 요청이면 `HttpServletRequest`를 **`MultipartHttpServletRequest`로 래핑**
4. 이후 컨트롤러에서 `@RequestParam MultipartFile`로 주입 가능

---

## 🔧 종류 및 특징

| 구현체                                | 설명                           | 비고                       |
| ---------------------------------- | ---------------------------- | ------------------------ |
| `StandardServletMultipartResolver` | 서블릿 3.0 기반 (스프링 부트 기본값)      | 가볍고 설정 간단                |
| `CommonsMultipartResolver`         | Apache Commons FileUpload 사용 | 상세 설정 가능 (인코딩, 업로드 제한 등) |

> ⚙️ 스프링 부트에서는 특별히 설정하지 않아도 `StandardServletMultipartResolver`가 자동 등록됨

---

## 🔧 커스터마이징 예시 (CommonsMultipartResolver)

```java
@Bean
public CommonsMultipartResolver multipartResolver() {
    CommonsMultipartResolver resolver = new CommonsMultipartResolver();
    resolver.setMaxUploadSize(10485760); // 10MB
    resolver.setDefaultEncoding(\"UTF-8\");
    return resolver;
}
```


## ✅ Q2. `MultipartFile`을 `@RequestParam`으로 받을 때와 `@ModelAttribute`로 받을 때의 차이점은?

| 구분       | `@RequestParam MultipartFile` | `@ModelAttribute SomeForm`       |
| -------- | ----------------------------- | -------------------------------- |
| 사용 목적    | 파일 단독 업로드                     | 파일 + 일반 입력값을 함께 수신               |
| 선언 위치    | 컨트롤러 파라미터에 직접 선언              | DTO 클래스 필드로 포함                   |
| 바인딩 방식   | name 속성 매칭                    | 객체 내 필드 이름 매칭                    |
| 사용성      | 단일 파일에 적합                     | 다중 필드 + 다중 파일에 적합                |
| 검증       | 수동 검증 필요                      | `@Valid`, `BindingResult`로 검증 가능 |
| 뷰 렌더링 활용 | 불편                            | DTO로 폼 재랜더링 용이                   |


---


## 💡 인사이트

* HTML 전송 방식에 따라 서버 파싱 방식이 완전히 달라진다. (multipart와 x-www-form-urlencoded)
* 스프링은 MultipartFile로 추상화하여 코드량을 줄이고 가독성을 크게 향상시킨다.

---

