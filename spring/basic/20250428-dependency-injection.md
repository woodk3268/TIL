# 의존관계 자동주입
## 1. 다양한 의존관계 주입 방법
### 1-1. 생성자 주입
- 생성자를 통해서 의존 관계를 주입 받는 방법이다.
- 특징
  - 생성자 호출 시점에 딱 1번만 호출되는 것이 보장된다.
  - 불변, 필수 의존관계에 사용
 
```
@Component
 public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
     this.memberRepository = memberRepository;
     this.discountPolicy = discountPolicy;
      }
 }
```
- 생성자가 딱 1개만 있으면 @Autowired를 생략해도 자동 주입된다. 물론 스프링 빈에만 해당한다.

### 1-2. 수정자 주입(setter 주입)
- setter라 불리는 필드의 값을 변경하는 수정자 메서드를 통해서 의존관계를 주입하는 방법이다.
- 특징
  - 선택, 변경 가능성이 있는 의존관계에 사용
```
@Component
 public class OrderServiceImpl implements OrderService {
     private MemberRepository memberRepository;
     private DiscountPolicy discountPolicy;

        @Autowired
         public void setMemberRepository(MemberRepository memberRepository) {
         this.memberRepository = memberRepository;
        }
        @Autowired
         public void setDiscountPolicy(DiscountPolicy discountPolicy) {
         this.discountPolicy = discountPolicy;
        }
     }
```

### 1-3. 필드 주입
- 필드에 바로 주입하는 방법이다.
- 특징
  - 외부에서 변경이 불가능해서 테스트하기 힘들다.
  - DI 프레임워크가 없으면 아무것도 할 수 없다.
 
```
@Component
 public class OrderServiceImpl implements OrderService {

     @Autowired
     private MemberRepository memberRepository;
     @Autowired
     private DiscountPolicy discountPolicy;
 }
```

### 1-4. 일반 메서드 주입
- 일반 메서드를 통해서 주입받을 수 있다.
- 특징
  - 한번에 여러 필드를 주입받을 수 있다.
  - 일반적으로 잘 사용하지 않는다.
 
```
@Component
 public class OrderServiceImpl implements OrderService {

    private MemberRepository memberRepository;
    private DiscountPolicy discountPolicy;

      @Autowired
     public void init(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
       this.memberRepository = memberRepository;
       this.discountPolicy = discountPolicy;
    }
 }
```

## 2. 옵션 처리
- 주입할 스프링 빈이 없어도 동작해야 할 때가 있다.
- 그런데 @Autowired만 사용하면 required 옵션의 기본값이 true로 되어 있어서 자동 주입 대상이 없으면 오류가 발생한다.
- 자동 주입 대상을 옵션으로 처리하는 방법은 다음과 같다.

```
 //호출 안됨
  @Autowired(required = false)
 public void setNoBean1(Member member) {
    System.out.println("setNoBean1 = " + member);
 }
 //null 호출
  @Autowired
 public void setNoBean2(@Nullable Member member) {
    System.out.println("setNoBean2 = " + member);
 }
 //Optional.empty 호출
  @Autowired(required = false)
 public void setNoBean3(Optional<Member> member) {
    System.out.println("setNoBean3 = " + member);
 }
```
- @Autowired(required = false) : 자동 주입할 대상이 없으면 수정자 메서드 자체가 호출 안된다.
- @Nullable : 자동주입할 대상이 없으면 null이 입력된다
- Optional<> : 자동주입할 대상이 없으면 Optional.empty가 입력된다.

## 3. 결론 - 생성자 주입을 선택하자.
**불변**
- 대부분의 의존관계 주입은 한번 일어나면 애플리케이션 종료시점까지 의존관계를 변경할 일이 없다.
- 오히려 대부분의 의존관계는 애플리케이션 종료 전까지 변하면 안된다.
- 수정자 주입을 사용하면 setXxx 메서드를 public으로 열어두어야 한다.
- 누군가 실수로 변경할 수도 있고, 변경하면 안 되는 메서드를 열어두는 것은 좋은 설계 방법이 아니다.

**누락**
```
 public class OrderServiceImpl implements OrderService {

     private MemberRepository memberRepository;
     private DiscountPolicy discountPolicy;

      @Autowired
     public void setMemberRepository(MemberRepository memberRepository) {
     this.memberRepository = memberRepository;
        }
     @Autowired
     public void setDiscountPolicy(DiscountPolicy discountPolicy) {
     this.discountPolicy = discountPolicy;
        }
     //...
 }
```
- @Autowired가 프레임워크 안에서 동작할 때는 의존관계가 없으면 오류가 발생하지만, 지금은 프레임워크 없이 순수한 자바 코드로만 단위 테스트를 수행하고 있다.

```
 @Test
 void createOrder() {
 OrderServiceImpl orderService = new OrderServiceImpl();
    orderService.createOrder(1L, "itemA", 10000);
 }
```
- 실행 결과는 NPE(Null Point Exception)이 발생하는데, memberRepository, discountPolicy 모두 의존관계 주입이 누락되었기 때문이다.
- 생성자 주입을 사용하면 주입 데이터를 누락했을 때 컴파일 오류가 발생한다.
  
**final 키워드**
- 생성자 주입을 사용하면 필드에 final 키워드를 사용할 수 있다.
- 그래서 생성자에서 혹시라도 값이 설정되지 않는 오류를 컴파일 시점에 막아준다.

```
@Component
 public class OrderServiceImpl implements OrderService {

  private final MemberRepository memberRepository;
  private final DiscountPolicy discountPolicy;

   @Autowired
   public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
   this.memberRepository = memberRepository;
    }
 //...
 }
```
- 필수 필드인 discountPolicy에 값을 설정해야 하는데, 이 부분이 누락되었다.
- 자바는 컴파일 시점에 다음 오류를 발생시킨다.
- java: variable discountPolicy might not have been initialized

## 4. 조회 빈이 2개 이상 - 문제
- @Autowired는 타입(Type)으로 조회한다.
```
 @Autowired
 private DiscountPolicy discountPolicy
```

-타입으로 조회하기 때문에, 마치 다음 코드와 유사하게 동작한다.
```
ac.getBean(DiscountPolicy.class)
```
- 타입으로 조회하면 선택된 빈이 2개 이상일 때 문제가 발생한다.

```
 @Component
 public class FixDiscountPolicy implements DiscountPolicy {}
```

```
@Component
 public class RateDiscountPolicy implements DiscountPolicy {}
```

- DiscountPolicy의 하위 타입인 FixDiscountPolicy, RateDiscountPolicy 둘 다 스프링 빈으로 선언해보자.
- 그리고 의존관계 자동주입을 실행하면, NoUniqueBeanDefinitionException 오류가 발생한다.
```
 @Autowired
 private DiscountPolicy discountPolicy
```
### 4-1. @Autowired 필드명 매칭
- @Autowired는 타입 매칭을 시도하고, 이때 여러 빈이 있으면 필드 이름, 파라미터 이름으로 빈 이름을 추가 매칭한다.
- 필드명을 빈 이름으로 변경
```
 @Autowired
 private DiscountPolicy rateDiscountPolicy
```
- 필드명이 rateDiscountPolicy이므로 정상 주입된다.
- 필드명 매칭은 먼저 타입 매칭을 시도하고 그 결과에 여러 빈이 있을 때 추가로 동작하는 기능이다.

**@Autowired 매칭 정리**
1. 타입 매칭
2. 타입 매칭의 결과가 2개 이상일 때 필드 명, 파라미터 명으로 빈 이름 매칭

### 4-2. @Qualifier 사용
- @Qualifier는 추가 구분자를 붙여주는 방법이다.
```
 @Component
 @Qualifier("mainDiscountPolicy")
 public class RateDiscountPolicy implements DiscountPolicy {}
```

```
@Component
@Qualifier("fixDiscountPolicy")
public class FixDiscountPolicy implements DiscountPolicy {}
```

```
 @Autowired
 public OrderServiceImpl(MemberRepository memberRepository,
                        @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
 this.memberRepository = memberRepository;
 this.discountPolicy = discountPolicy;
 }
```

**@Qualifier 정리**
1. @Qualifier끼리 매칭
2. 빈 이름 매칭
3. NoSuchBeanDefinitionException 예외 발생

### 4-3.  @Primary 사용
- @Primary는 우선순위를 정하는 방법이다. @Autowired 시에 여러 빈이 매칭되면 @Primary가 우선권을 가진다.

```
 @Component
 @Primary
 public class RateDiscountPolicy implements DiscountPolicy {}

 @Component
 public class FixDiscountPolicy implements DiscountPolicy {}
```

```
 @Autowired
 public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
  
   this.memberRepository = memberRepository;
   this.discountPolicy = discountPolicy;
 }
```

## 5. 자동, 수동의 올바른 실무 운영 기준
- 업무 로직에 사용되는 객체(웹을 지원하는 컨트롤러, 핵심 비즈니스 로직이 있는 서비스, 데이터 계층의 로직을 처리하는 리포지토리 등)는 편리한 자동 기능을 기본으로 사용
- 애플리케이션에 광범위하게 영향을 미치는 기술 지원 객체(DB 연결, 공통 로그 처리)는 수동 등록
- 다형성을 적극 활용하는 비즈니스 로직은 수동 등록 고민
