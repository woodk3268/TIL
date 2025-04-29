# 빈 생명주기 콜백
- DB 커넥션 풀이나 네트워크 소켓처럼 애플리케이션 시작 시점에 필요한 연결을 미리 해두고,
   애플리케이션 종료 시점에 연결을 모두 종료하는 작업을 진행하려면, 객체의 초기화와 종료 작업이 필요하다.
- 스프링을 통해 이러한 초기화 작업과 종료 작업을 어떻게 진행하는지 살펴보자.

## 1. 스프링의 빈 생명주기 콜백 적용 전 예제
```
public class NetworkClient {

 private String url;

 public NetworkClient() {
 System.out.println("생성자 호출, url = " + url);
 connect();
 call("초기화 연결 메시지");
}

public void setUrl(String url) {
 this.url = url;
    }

 //서비스 시작시 호출
public void connect() {
 System.out.println("connect: " + url);
    }

 public void call(String message) {
 System.out.println("call: " + url + " message = " + message);
    }

 //서비스 종료시 호출
public void disconnect() {
 System.out.println("close: " + url);
    }
 }
```

**스프링 환경설정과 실행**
```
public class BeanLifeCycleTest {

  @Test
 public void lifeCycleTest() {
     ConfigurableApplicationContext ac = new AnnotationConfigApplicationContext(LifeCycleConfig.class);
     NetworkClient client = ac.getBean(NetworkClient.class);
     ac.close(); //스프링 컨테이너를 종료, ConfigurableApplicationContext 필요
    }

  @Configuration
 static class LifeCycleConfig {

      @Bean
     public NetworkClient networkClient() {
         NetworkClient networkClient = new NetworkClient();
         networkClient.setUrl("http://hello-spring.dev");
         return networkClient;
            }
    }
 }
```

**실행 결과**
- 생성자 부분을 보면 url 정보 없이 connect가 호출되는 것을 알 수 있다.
- 객체를 생성하는 단계에는 url이 없고, 객체를 생성한 다음에 외부에서 수정자 주입을 통해서 setUrl()이 호출되어야 url이 존재하게 된다.

## 2. 스프링 빈의 라이프사이클
- 스프링 빈은 객체를 생성하고 의존관계 주입이 다 끝난 다음에야 필요한 데이터를 사용할 수 있는 준비가 완료된다.
- 따라서 초기화 작업은 의존관계 주입이 모두 완료되고 난 다음에 호출해야 한다.
- **스프링은 의존관계 주입이 완료되면 스프링 빈에게 콜백 메서드를 통해서 초기화 시점을 알려주는 다양한 기능을 제공한다.**
- **스프링은 스프링 컨테이너가 종료되기 직전에 소멸 콜백을 준다.**

**스프링 빈의 이벤트 사이클**
- **스프링 컨테이너 생성 -> 스프링 빈 생성 -> 의존관계 주입 -> 초기화 콜백 -> 사용 -> 소멸 전 콜백 -> 스프링 종료**

## 3. 스프링이 빈 생명주기 콜백을 지원하는 방법
### 3-1. 인터페이스 InitializingBean, DisposableBean

```
 public class NetworkClient implements InitializingBean, DisposableBean {
...

  @Override
 public void afterPropertiesSet() throws Exception {
     connect();
     call("초기화 연결 메시지");
    }

  @Override
 public void destroy() throws Exception {
     disConnect();
    }
 }
```

- InitializingBean은 afterPropertiesSet() 메서드로 초기화를 지원한다.
- DisposableBean은 destroy() 메서드로 소멸을 지원한다.

**단점**
- 이 인터페이스는 스프링 전용 인터페이스다. 해당 코드가 스프링 전용 인터페이스에 의존한다.
- 초기화, 소멸 메서드의 이름을 변경할 수 없다.
- 내가 코드를 고칠 수 없는 외부 라이브러리에 적용할 수 없다.

### 3-2. 빈 등록 초기화, 소멸 메서드 지정

```
 public class NetworkClient {
...
     public void init() {
         System.out.println("NetworkClient.init");
         connect();
         call("초기화 연결 메시지");
        }
     public void close() {
         System.out.println("NetworkClient.close");
         disConnect();
        }
 }
```

```
 @Configuration
 static class LifeCycleConfig {

        @Bean(initMethod = "init", destroyMethod = "close")
     public NetworkClient networkClient() {

          NetworkClient networkClient = new NetworkClient();
          networkClient.setUrl("http://hello-spring.dev");
          return networkClient;
        }
 }
```
- 설정 정보에 @Bean(initMethod = "init", destroyMethod = "close") 처럼 초기화, 소멸 메서드를 지정할 수 있다.

**특징**
- 메서드 이름을 자유롭게 줄 수 있다.
- 스프링 빈이 스프링 코드에 의존하지 않는다.
- 설정 정보를 사용하기 때문에 코드를 고칠 수 없는 외부 라이브러리에도 초기화, 종료 메서드를 적용할 수 있다.

### 3-3. 애노테이션 @PostConstruct, @PreDestroy
```
 public class NetworkClient {

  @PostConstruct
 public void init() {
     System.out.println("NetworkClient.init");
     connect();
     call("초기화 연결 메시지");
    }

  @PreDestroy
 public void close() {
     System.out.println("NetworkClient.close");
     disConnect();
    }
 }
```
- @PostConstruct, @PreDestroy 이 두 애노테이션을 사용하면 가장 편리하게 초기화와 종료를 실행할 수 있다.

**특징**
- 패키지를 보면 javax.annotation.PostConstruct이다. 스프링에 종속적인 기술이 아니라 자바 표준이다. 따라서 스프링이 아닌 다른 컨테이너에서도 동작한다.
- 유일한 단점은 외부 라이브러리에는 적용하지 못한다는 것이다.

## 4. 정리
- **@PostConstruct, @PreDestroy 애노테이션을 사용하자**
- 코드를 고칠 수 없는 외부 라이브러리를 초기화, 종료해야 하면 @Bean의 initMethod, destroyMethod를 사용하자
