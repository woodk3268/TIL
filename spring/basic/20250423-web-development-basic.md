# 스프링 웹 개발 기초

## 정적 컨텐츠
resources/static/hello-static.html
```
 <!DOCTYPE HTML>
 <html>
 <head>
    <title>static content</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
 </head>
 <body>
정적 컨텐츠 입니다.
 </body>
 </html>
```
1. 웹 브라우저가 http://localhost:8080/hello-static.html 요청
2. 내장 톰캣 서버 수신 -> pring 컨테이너에게 해당 요청을 처리할 수 있는 컨트롤러가 있는지 물어봄
   - resources/static/hello-static.html 경로에 있는 실제 파일 찾음
   - 해당 파일이 존재하면 Spring Boot가 이 파일을 그대로 브라우저에게 응답
   - 즉, 정적 리소스는 컨트롤러 없이도 바로 응답 가능
      

## MVC와 템플릿 엔진
- MVC : Model, View, Controller

**Controller**
```
@Controller
public class HelloController {

    @GetMapping("hello-mvc")
     public String helloMvc(@RequestParam("name") String name, Model model) {
        model.addAttribute("name", name);
        return "hello-template";
    }
 }
```

**View**
resources/templates/hello-template.html
```
<html xmlns:th="http://www.thymeleaf.org">
 <body>
 <p th:text="'hello ' + ${name}">hello! empty</p>
 </body>
 </html>
```
1. 웹 브라우저가 [http://localhost:8080/hello-static.html](http://localhost:8080/hello-mvc?name=spring) 요청
2. 내장 톰캣 서버 수신 -> pring 컨테이너에게 해당 요청을 처리할 수 있는 컨트롤러가 있는지 물어봄
   - helloController가 해당 URL을 매핑해서 실행
   - viewResolver가 templates/hello-template.html 파일 찾아서 렌더링

## API
**ResponseBody 문자 반환**
```
@Controller
public class HelloController {
 @GetMapping("hello-api")
 @ResponseBody
 public Hello helloApi(@RequestParam("name") String name) {
   Hello hello = new Hello();
   hello.setName(name);
   return hello;
 }
}
```
- @ResponseBody를 사용하고, 객체를 반환하면 객체가 JSON으로 변환된다.

**ResponseBody 사용 원리**
- @ResponseBody 사용
  - HTTP의 BODY에 문자 내용을 직접 반환
  - viewResolver 대신에 HttpMessageConverter가 동작
