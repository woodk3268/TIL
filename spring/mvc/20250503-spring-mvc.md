# 스프링 MVC
## 1. 구조 이해
- 스프링 MVC도 프론트 컨트롤러 패턴으로 구현되어 있다.
- 스프링 MVC의 프론트 컨트롤러가 바로 디스패처 서블릿이다.

### 1-1. DisPatcherServlet 서블릿 등록
- DisPatcherServlet도 부모 클래스에서 HttpServlet을 상속 받아서 사용하고, 서블릿으로 동작한다.
  - DispatcherServlet -> FrameworkServlet -> HttpServletBean -> HttpServlet
- 스프링 부트는 DisPatcherServlet을 서블릿으로 자동 등록하면서 모든 경로(urlPatterns="/")에 대해서 매핑한다.

### 1-2. 요청 흐름
- 서블릿이 호출되면 HttpServlet이 제공하는 service()가 호출된다.
- 스프링 MVC는 DisPatcherServlet의 부모인 FrameworkServlet에서 service()를 오버라이딩 해두었다.
- FrameworkServlet.service()를 시작으로 여러 메서드가 호출되면서 DispatcherServlet.doDispatch()가 호출된다.

### 1-3. 동작 순서
1) **핸들러 조회** : 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다.
2) **핸들러 어댑터 조회** : 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
3) **핸들러 어댑터 실행** : 핸들러 어댑터를 실행한다.
4) **핸들러 실행** : 핸들러 어댑터가 실제 핸들러를 실행한다.
5) **ModelAndView 반환** : 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환한다.
6) **ViewResolver 호출** : 뷰 리졸버를 찾고 실행한다.
7) **View 반환** : 뷰 리졸버는 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환한다.
8) **뷰 렌더링** : 뷰를 통해서 뷰를 렌더링한다.

## 2. 핸들러 매핑과 핸들러 어댑터
컨트롤러가 호출되려면 다음 2가지가 필요하다.
- **HandlerMapping(핸들러 매핑)**
  - 핸들러 매핑에서 컨트롤러를 찾을 수 있어야 한다.
- **HandlerAdapter(핸들러 어댑터)**
   - 핸들러 매핑을 통해서 찾은 핸들러를 실행할 수 있는 핸들러 어댑터가 필요하다.

 - **스프링 부트가 자동 등록하는 핸들러 매핑과 핸들러 어댑터**
   - HandlerMapping
      0 = RequestMappingHandlerMapping   : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
      1 = BeanNameUrlHandlerMapping      : 스프링 빈의 이름으로 핸들러를 찾는다.
   - HandlerAdapter
      0 = RequestMappingHandlerAdapter   : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
      1 = HttpRequestHandlerAdapter      : HttpRequestHandler 처리
      2 = SimpleControllerHandlerAdapter : Controller 인터페이스(애노테이션X, 과거에 사용) 처리

  - **동작 방식**
    1. HandlerMapping 을 순서대로 실행해서, 핸들러를 찾는다.
    2. HandlerAdapter의 supports()를 순서대로 호출한다.
    3. 디스패처 서블릿이 조회한 어댑터를 실행하면서 핸들러 정보도 함께 넘겨준다. 어댑터는 핸들러를 내부에서 실행하고, 그 결과를 반환한다.

## 3. 뷰 리졸버
- ModelAndView의 **뷰 이름(String)**을 받아서, 최종적으로 렌더링할 View 객체로 매핑하는 역할을 한다.

- **스프링 부트가 자동 등록하는 뷰 리졸버**
 1 = BeanNameViewResolver         : 빈 이름으로 뷰를 찾아서 반환한다.
 2 = InternalResourceViewResolver : JSP를 처리할 수 있는 뷰를 반환한다.

- **동작 방식**
  1. 핸들러 어댑터 호출을 통해 논리 뷰 이름을 획득한다.
  2. 논리 뷰 이름으로 viewResolver를 순서대로 호출한다.
  3. 스프링 빈으로 등록된 뷰가 없으면, InternalResourceViewResolver가 호출된다.
  4. View를 반환한다.
  5. view.render()가 호출되고 InternalResourceViewResolver는 forward()를 사용해서 JSP를 실행한다.
 
## 4. 스프링 MVC 시작
- @Controller : 스프링이 자동으로 스프링 빈으로 등록한다. 스프링 MVC에서 애노테이션 기반 컨트롤러로 인식한다.
- @RequestMapping : 요청 정보를 매핑한다. 해당 URL이 호출되면 이 메서드가 호출된다.
- ModelAndView : 모델과 뷰 정보를 담아서 반환한다.
