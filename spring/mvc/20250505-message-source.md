# 학습 주제 : Spring Boot에서 메시지 관리 및 국제화(i18n) 기능 사용 방법

## 1. 메시지 기능(MessageSource)
- messages.properties 파일을 만들어 label, 버튼명 등을 한 곳에서 관리
- 하드코딩 대신 key 기반 참조 → 유지보수성 향상
- 타임리프에서 #{key} 표현식 사용
  - 예: <label th:text="#{item.itemName}">상품명</label>

## 2. 국제화(i18n) 적용
- 언어별 메시지 파일 생성:
  - messages.properties (기본)
  - messages_ko.properties (한국어)
  - messages_en.properties (영어)
- 클라이언트 요청의 Accept-Language 헤더를 기준으로 메시지 파일 자동 선택

**i18n의 주요 구성 요소**

| 구성 요소            | 설명                                                                 |
|---------------------|----------------------------------------------------------------------|
| 🌐 **Locale**        | 언어와 국가를 조합한 설정 (예: `en_US`, `ko_KR`)                     |
| 📂 **Resource Bundle** | 언어별 메시지를 저장한 파일 (`messages_ko.properties`, `messages_en.properties`) |
| 🧠 **Message Resolver** | 현재 사용자의 Locale을 기준으로 알맞은 메시지를 선택하는 로직           |
| 💬 **UI 텍스트 분리**   | 소스 코드나 템플릿에서 직접 문자열을 쓰지 않고 키를 통해 참조              |


## 3. 스프링 메시지 설정 방법
(1) 수동 등록
```
@Bean
public MessageSource messageSource() {
    ResourceBundleMessageSource source = new ResourceBundleMessageSource();
    source.setBasenames("messages", "errors");
    source.setDefaultEncoding("UTF-8");
    return source;
}

```

(2) Spring Boot 자동 설정
- application.properties
```
spring.messages.basename=messages
```

## 4. 테스트 코드 사용 예시
- messages.properties
```
hello=안녕
hello.name=안녕 {0}
```

```
String result = ms.getMessage("hello", null, null); 
// → 기본 Locale 기준 메시지 (e.g. "안녕")
```

```
String result = ms.getMessage("hello.name", new Object[]{"Spring"}, null); 
// → "안녕 Spring"
```

## 5. Locale 처리 방식
| 처리 방식 | 설명                                               |
|------------|----------------------------------------------------|
| 기본       | Accept-Language 헤더를 기반으로 메시지 파일 선택  |
| 커스텀     | LocaleResolver를 재정의해 쿠키, 세션 등으로 언어 선택 가능 |

# 인사이트
- View 단 텍스트를 key 기반으로 관리하면 UI 변경 대응력이 크게 향상됨
- Locale 변경 UI를 사용자에게 제공할 경우 LocaleResolver 커스터마이징 고려
