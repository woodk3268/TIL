# 컴포넌트 스캔

## 1. 컴포넌트 스캔과 의존관계 자동 주입 시작하기
- 스프링 빈을 직접 등록할 때는 자바 코드의 @Bean이나 XML의 <bean> 등을 통해서 설정 정보에 직접 등록할 스프링 빈을 나열한다.
- 이렇게 등록해야할 스프링 빈이 수십, 수백 개가 되면 일일이 등록하기가 귀찮다.
- 그래서 스프링은 설정 정보가 없어도 자동으로 스프링 빈을 등록하는 컴포넌트 스캔이라는 기능을 제공한다.
- 또 의존관계도 자동으로 주입하는 @Autowired 라는 기능도 제공한다.

```
@Configuration
@ComponentScan
 public class AutoAppConfig {
 }
```
- 컴포넌트 스캔을 사용하려면 먼저 @ComponentScan을 설정 정보에 붙여주면 된다.
- 기존의 AppConfig 와는 다르게 @Bean으로 등록한 클래스가 하나도 없다.
- @Configuration 소스코드에도 @Component 애노테이션이 붙어 있으므로, 컴포넌트 스캔의 대상이 된다.

```
@Component
public class MemberServiceImpl implements MemberService {
    private final MemberRepository memberRepository;
 
    @Autowired
    public MemberServiceImpl(MemberRepository memberRepository) {
      this.memberRepository = memberRepository;
    }
```
- @Autowired는 의존관계를 자동으로 주입해준다.

```
  @Test
 void basicScan() {
     ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class);
     MemberService memberService = ac.getBean(MemberService.class);
     assertThat(memberService).isInstanceOf(MemberService.class);
    }
```

## 2. 컴포넌트 스캔과 자동 의존관계 주입 동작 방식
### 2-1. @ComponentScan
- @ComponentScan은 @Component가 붙은 모든 클래스를 스프링 빈으로 등록한다.
- 이때 스프링 빈의 기본 이름은 클래스 명을 사용하되 맨 앞글자만 소문자를 사용한다.
  - 빈 이름 기본 전략 : MemberServiceImpl 클래스 -> memberServiceImpl
  - 빈 이름 직접 지정 : 만약 스프링 빈의 이름을 직접 지정하고 싶으면 `@Component("memberService2")` 이런식으로 이름을 부여하면 된다.
 
### 2-2. @Autowired 의존관계 자동 주입
- 생성자가 @Autowired를 지정하면, 스프링 컨테이너가 자동으로 해당 스프링 빈을 찾아서 주입한다.
- 이때 기본 조회 전략은 타입이 같은 빈을 찾아서 주입한다.
   - getBean(MemberRepository.class)와 동일하다고 이해하면 된다.
 
## 3. 탐색 위치와 기본 스캔 대상
### 3-1. 탐색할 패키지의 시작 위치 지정
: 필요한 위치부터 탐색하도록 시작 위치를 지정할 수 있다.
```
@ComponentScan(
        basePackages = "hello.core",
 }
```
- basePackages : 탐색할 패키지의 시작 위치를 지정한다. 이 패키지를 포함해서 하위 패키지를 모두 탐색한다.
- basePackageClasses : 지정한 클래스의 패키지를 탐색 시작 위치로 지정한다.
- 만약 지정하지 않으면 @ComponentScan이 붙은 설정 정보 클래스의 패키지가 시작 위치가 된다.
- 참고로 스프링 부트를 사용하면 스프링 부트의 대표 시작 정보인 @SpringBootApplication을 이 프로젝트 시작 루트 위치에 두는 것이 관례이다.
- 그리고 이 설정 안에 바로 @ComponentScan이 들어있다.

### 3-2. 컴포넌트 스캔 기본 대상
- @Component : 컴포넌트 스캔에서 사용
- @Controller : 스프링 MVC 컨트롤러에서 사용
- @Service : 스프링 비즈니스 로직에서 사용. 특별한 처리를 하지 않음.
- @Repository : 스프링 데이터 접근 계층에서 사용. 데이터 계층의 예외를 스프링 예외로 변환해줌
- @Configuration : 스프링 설정 정보에서 사용. 스프링 빈이 싱글톤을 유지하도록 추가 처리

@Component 외의 애노테이션의 소스코드를 보면 @Component를 포함하고 있다.

### 3-3. 필터
- includeFilters: 컴포넌트 스캔 대상을 추가로 지정한다.
- excludeFilters : 컴포넌트 스캔에서 제외할 대상을 지정한다.

**컴포넌트 스캔 대상에 추가할 애노테이션**
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyIncludeComponent {
}
```

**컴포넌트 스캔 대상에서 제외할 애노테이션**
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyExcludeComponent {
}
```

**컴포넌트 스캔 대상에 추가할 클래스**
```
@MyIncludeComponent
public class BeanA {
}
```
**컴포넌트 스캔 대상에서 제외할 애노테이션**
```
@MyExcludeComponent
public class BeanB {
}
```

**설정 정보와 전체 테스트 코드**
```
   @Test
   void filterScan() {
       ApplicationContext ac = new AnnotationConfigApplicationContext(ComponentFilterAppConfig.class);

       BeanA beanA = ac.getBean("beanA", BeanA.class);
       assertThat(beanA).isNotNull();

       Assertions.assertThrows( NoSuchBeanDefinitionException.class,() -> ac.getBean("beanB", BeanB.class));
    }
    @Configuration
    @ComponentScan(
            includeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class),
            excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyExcludeComponent.class)
    )
     static class ComponentFilterAppConfig {
    }
```
- includeFilters에 MyIncludeComponent 애노테이션을 추가해서 BeanA가 스프링 빈에 등록된다.
- excludeFilters에 MyExcludeComponent 애노테이션을 추가해서 BeanB는 스프링 빈에 등록되지 않는다.

## 4. 중복 등록과 충돌
- 컴포넌트 스캔에서 같은 빈 이름을 등록하면 어떻게 될까?
- 다음 두가지 상황이 있다.
   - 자동 빈 등록 vs 자동 빈 등록
   - 수동 빈 등록 vs 자동 빈 등록
 
### 4-1. 자동 빈 등록 vs 자동 빈 등록
- 컴포넌트 스캔에 의해 자동으로 스프링 빈이 등록되는데, 그 이름이 같은 경우 스프링은 오류를 발생시킨다.
  - ConflictingBeanDefinitionException 예외 발생

### 4-2. 수동 빈 등록 vs 자동 빈 등록
```
@Component
 public class MemoryMemberRepository implements MemberRepository {}
```

```
 @Configuration
 @ComponentScan(
        excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = Configuration.class)
 )
 public class AutoAppConfig {
     @Bean(name = "memoryMemberRepository")
     public MemberRepository memberRepository() {
         return new MemoryMemberRepository();
      }
 }
```
- 이 경우 수동 빈 등록이 우선권을 가진다.(수동 빈이 자동 빈을 오버라이딩 해버린다.)
- 만약 개발자가 의도한 결과가 아니라면, 정말 잡기 어려운 버그가 만들어진다.
- 그래서 최근 스프링 부트에서는 수동 빈 등록과 자동 빈 등록이 충돌나면 오류가 발생하도록 기본값을 바꾸었다.
