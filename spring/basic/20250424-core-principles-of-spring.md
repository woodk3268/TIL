# 스프링 핵심원리 

## 1. 예제
### 1-1. 예제 코드
```
public class OrderServiceImpl implements OrderService {
 //    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
 private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
 }
```
- 클래스 의존관계를 분석해보면, 주문서비스 클라이언트(OrderServiceImpl)는 DiscountPolicy 인터페이스뿐만 아니라 구체(구현)클래스에도 의존하고 있다.
  - 추상(인터페이스) 의존 : DiscountPolicy
  - 구체(구현) 클래스 : FixDiscountPolicy, RateDiscountPolicy
- OCP 위반 : 기능을 확장해서 변경하면, 클라이언트 코드에 영향을 준다. 따라서 OCP를 위반한다.

### 1-2. 해결 방안
- DIP를 위반하지 않도록 인터페이스에만 의존하도록 설계를 변경한다.
```
 public class OrderServiceImpl implements OrderService {
 //private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
 private DiscountPolicy discountPolicy;
 }
```
- 그런데 구현체가 없는데 어떻게 코드를 실행할 수 잇을까?
- 이 문제를 해결하려면 누군가가 클라이언트인 OrderServiceImpl에 DiscountPolicy의 구현 객체를 대신 생성하고 주입해주어야 한다.

## 2. 관심사의 분리
- 애플리케이션을 하나의 공연으로, 각각의 인터페이스를 배역으로 생각하자.
- 배우는 본인의 역할인 배역을 수행하는 것에만 집중해야 한다.
- 남자 주인공 구현체는 어떤 여자주인공이 선택되더라도 똑같이 공연을 할 수 있어야 한다.
- 공연을 구성하고, 담당 배우를 섭외하고, 역할에 맞는 배우를 지정하는 책임을 담당하는 별도의 공연 기획자가 나올 시점이다.
- 공연 기획자를 만들고, 배우와 공연 기획자의 책임을 확실히 분리하자.

## 3. AppConfig 등장
- 애플리케이션의 전체 동작 방식을 구성(config)하기 위해, 구현 객체를 생성하고, 연결하는 책임을 가지는 별도의 설정 클래스를 만들자.
```
public class AppConfig {
 public MemberService memberService() {
   return new MemberServiceImpl(memberRepository());
    }
 public OrderService orderService() {
   return new OrderServiceImpl(memberRepository(), discountPolicy());
    }
 public MemberRepository memberRepository() {
   return new MemoryMemberRepository();
    }
 public DiscountPolicy discountPolicy() {
  // return new FixDiscountPolicy();
   return new RateDiscountPolicy();

    }
}
```
- 객체의 생성과 연결은 AppConfig가 담당한다.
- DIP 완성 : MemberServiceImpl은 MemberRepository인 추상에만 의존하면 된다. 이제 구체 클래스를 몰라도 된다.
- 관심사의 분리 : 객체를 생성하고 연결하는 역할과 실행하는 역할이 명확하게 분리되었다.
- appConfig 객체는 memoryMemberRepository 입장에서 보면 의존관계를 마치 외부에서 주입해주는 것 같다고 해서 DI(Dependency Injection)이라고 한다.

**정리**
- AppConfig를 통해서 관심사를 확실하게 분리했다.
- AppConfig는 구체 클래스를 선택한다. 애플리케이션이 어떻게 동작해야할지 전체 구성을 책임진다.
- 각 구체 클래스는 기능을 실행하는 책임만 지면 된다.
- 애플리케이션이 크게 사용 영역과, 객체를 생성하고 구성하는 영역으로 분리되었다.
- 이제 할인 정책을 변경해도, 애플리케이션의 구성 역할을 담당하는 AppConfig만 변경하면 된다.
- 사용 영역의 어떤 코드도 변경할 필요가 없다.
