# 싱글톤 컨테이너
## 1. 웹 애플리케이션과 싱글톤
- 대부분의 스프링 애플리케이션은 웹 애플리케이션이다.
- 웹 애플리케이션은 보통 여러 고객이 동시에 요청을 한다.

```
    @Test
    @DisplayName("스프링 없는 순수한 DI 컨테이너")
     void pureContainer() {
        AppConfig appConfig = new AppConfig();
        MemberService memberService1 = appConfig.memberService();
        MemberService memberService2 = appConfig.memberService();
  
        assertThat(memberService1).isNotSameAs(memberService2);
    }
```

- 스프링 없는 순수한 DI 컨테이너인 AppConfig는 요청을 할 때마다 객체를 새로 생성한다.
- 고객 트래픽이 초당 100이 나오면 초당 100개 객체가 생성되고 소멸된다 -> 메모리 낭비가 심하다.
- 해결방안은 해당 객체가 딱 1개만 생성되고, 공유하도록 설계하면 된다. -> 싱글톤 패턴

## 2. 싱글톤 패턴
- 클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴이다.
- 그래서 객체 인스턴스를 2개 이상 생성하지 못하도록 막아야 한다.
- private 생성자를 사용해서 외부에서 임의로 new 키워드를 사용하지 못하도록 막아야 한다.

```
public class SingletonService {
     private static final SingletonService instance = new SingletonService();
     
     public static SingletonService getInstance() {
         return instance;
      }
     private SingletonService() {
      }
```
- 1. static 영역에 객체 instance를 미리 하나 생성해서 올려둔다.
  2. 이 객체 인스턴스가 필요하면 오직 getInstance() 메서드를 통해서만 조회할 수 있다. 이 메서드를 호출하면 항상 같은 인스턴스를 반환한다.
  3. 딱 1개의 객체 인스턴스만 존재해야 하므로, 생성자를 private으로 막아서 혹시라도 외부에서 new 키워드로 객체 인스턴스가 생성되는 것을 막는다.

```
@Test
@DisplayName("싱글톤 패턴을 적용한 객체 사용")
public void singletonServiceTest() {
      SingletonService singletonService1 = SingletonService.getInstance();
      SingletonService singletonService2 = SingletonService.getInstance();

      assertThat(singletonService1).isSameAs(singletonService2);
 }
```
- 호출할 때마다 같은 객체 인스턴스를 반환한다.

**문제점**
- 싱글톤 패턴을 구현하는 코드 자체가 많이 들어간다.
- 의존관계상 클라이언트가 구체 클래스에 의존한다 -> DIP를 위반한다.
- 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높다.

## 3. 싱글톤 컨테이너
- 스프링 컨테이너는 싱글톤 패턴을 적용하지 않아도, 객체 인스턴스를 싱글톤으로 관리한다.
- 스프링 컨테이너는 싱글톤 컨테이너 역할을 한다. 이렇게 싱글톤 객체를 생성하고 관리하는 기능을 싱글톤 레지스트리라 한다.
- 스프링 컨테이너의 이런 기능 덕분에 싱글톤 패턴의 모든 단점을 해결하면서 객체를 싱글톤으로 유지할 수 있다
  - 싱글톤 패턴을 위한 지저분한 코드가 들어가지 않아도 된다.
  - DIP, OCP, 테스트, private 생성자로부터 자유롭게 싱글톤을 사용할 수 있다.
 
```
 @Test
 @DisplayName("스프링 컨테이너와 싱글톤")
     void springContainer() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
        MemberService memberService1 = ac.getBean("memberService", MemberService.class);
        MemberService memberService2 = ac.getBean("memberService", MemberService.class);

        assertThat(memberService1).isSameAs(memberService2);
 }
```
- 스프링 컨테이너 덕분에 이미 만들어진 객체를 공유해서 효율적으로 재사용할 수 있다.

## 4. 싱글톤 방식의 주의점
- 객체 인스턴스를 하나만 생성해서 공유하는 싱글톤 방식은 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유하기 때문에 싱글톤 객체는 상태를 유지(stateful)하게 설계하면 안된다.
- 무상태(stateless)로 설계해야 한다.
  - 특정 클라이언트에 의존적인 필드가 있으면 안된다.
  - 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다.
  - 가급적 읽기만 가능해야 한다.
  - 필드 대신에 자바에서 공유되지 않는 지역변수, 파라미터, ThreadLocal 등을 사용해야 한다.
 
```
public class StatefulService {
      private int price; //상태를 유지하는 필드

      public void order(String name, int price) {
       this.price = price; //여기가 문제!
          }
      public int getPrice() {
       return price;
          }
 }
```

```
 @Test
 void statefulServiceSingleton() {
     ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
     StatefulService statefulService1 = ac.getBean("statefulService", StatefulService.class);
     StatefulService statefulService2 = ac.getBean("statefulService", StatefulService.class);

     statefulService1.order("userA", 10000);
     statefulService2.order("userB", 20000);

     int price = statefulService1.getPrice();
     Assertions.assertThat(statefulService1.getPrice()).isEqualTo(20000);
      }
```
-  StatefulService 의 price필드는 공유되는 필드인데, 특정 클라이언트가 값을 변경한다.
-  공유필드는 조심해야 한다! 스프링 빈은 항상 무상태(stateless)로 설계하자.


## 5. @Configuration과 싱글톤
```
 @Configuration
 public class AppConfig {
      @Bean
       public MemberService memberService() {
       return new MemberServiceImpl(memberRepository());
          }
      @Bean
       public OrderService orderService() {
       return new OrderServiceImpl(memberRepository(), discountPolicy());
          }
      @Bean
       public MemberRepository memberRepository() {
       return new MemoryMemberRepository();
      }
      ...
   }
```
- memberService, OrderService 빈을 만드는 코드를 보면 memberRepository()를 호출한다.
  - 이 메서드를 호출하면 new MemoryMemberRepository()를 호출한다.
- 결과적으로 각각 다른 2개의 MemoryMemberRepository가 생성되면서 싱글톤이 깨지는 것처럼 보인다.
- 스프링 컨테이너는 이 문제를 어떻게 해결할까?

## 6.  @Configuration과 바이트코드 조작의 마법
- 스프링 컨테이너는 싱글톤 레지스트리다.
- 따라서 스프링 빈이 싱글톤이 되도록 보장해주어야 한다.
- 자바 코드를 보면 분명 new MemoryMemberRepository()가 3번 호출되어야 하는 것이 맞다.
- 그래서 스프링은 클래스의 바이트코드를 조작하는 라이브러리를 사용한다.

```
 @Test
 void configurationDeep() {
     ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

     AppConfig bean = ac.getBean(AppConfig.class);
 }
```
- 사실 AnnotationConfigApplicationContext에 파라미터로 넘긴 값은 스프링 빈으로 등록된다.
- 그래서 AppConfig 도 스프링 빈이 된다.
- AppConfig가 순수한 클래스라면 클래스 정보가 다음과 같아야 한다.
  - `class hello.core.AppConfig`
- 그러나 예상과는 다르게 클래스명에 xxxCGLIB가 붙으면서 상당히 복잡해진 것을 볼 수 있다.
  - `class hello.core.AppConfig$$EnhancerBySpringCGLIB$$bd479d70`
- 스프링이 CGLIB라는 바이트코드 조작 라이브러리를 사용해서 AppConfig 클래스를 상속받은 임의의 다른 클래스를 만들고, 그 다른 클래스를 스프링 빈으로 등록한 것이다.
- 그 임의의 다른 클래스가 바로 싱글톤이 보장되도록 해준다.

 AppConfig@CGLIB 예상 코드
```
@Bean
 public MemberRepository memberRepository() {
     if (memoryMemberRepository가 이미 스프링 컨테이너에 등록되어 있으면?) {
          return 스프링 컨테이너에서 찾아서 반환;
      } else { //스프링 컨테이너에 없으면
          기존 로직을 호출해서 MemoryMemberRepository를 생성하고 스프링 컨테이너에 등록
          return 반환
```
- @Bean이 붙은 메서드마다 이미 스프링 빈이 존재하면 존재하는 빈을 반환하고, 스프링 빈이 없으면 생성해서 스프링 빈으로 등록하고 반환하는 코드가 동적으로 만들어진다.
- 덕분에 싱글톤이 보장되는 것이다.
