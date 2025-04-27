# 스프링 컨테이너 - BeanFactory와 ApplicationContext

## 1. BeanFactory
- 스프링 컨테이너의 최상위 인터페이스다.
- 스프링 빈을 관리하고 조회하는 역할을 담당한다.
- getBean()을 제공한다.

 ## 2. ApplicationContext
 - BeanFactory의 기능을 모두 상속받아서 제공한다.
 - 빈을 관리하고 조회하는 기능 외에 부가기능을 제공한다.
     - 메시지소스를 활용한 국제화 기능
     - 환경변수
     - 애플리케이션 이벤트
     - 편리한 리소스 조회

## 3. 정리
- ApplicationContext는 BeanFactory의 기능을 상속받는다.
- ApplicationContext는 빈 관리 기능 + 편리한 부가 기능을 제공한다.

---

# 스프링 빈 설정 메타 정보 - BeanDefinition
- BeanDefinition을 빈 설정 메타정보라 한다.
- @Bean, <bean> 당 각각 하나씩 메타 정보가 생성된다.
- 스프링 컨테이너는 이 메타정보를 기반으로 스프링 빈을 생성한다.
