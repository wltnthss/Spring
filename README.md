# Spring 개념 정리

## Spring 헷갈리는 개념 정리

**servlet-context.xml, root-context.xml 차이**

* 수업 도중에 옆에 있던 동료가 spring legacy project 생성시 생성되는 servlet-context.xml, root-context.xml 차이점에 대해서 물어봤다.
* servlet-context는 controller 관련 로직, root-context는 service, repository 관련 빈 등록? 이라고만 알았는데 정확한 동작원리는 알고있지 못하기에 정리하고자 한다.

**동작원리**

1. context-param을 통해 root-context 설정을 합니다.
2. listener 태그의 ContextLoaderListener 클래스를 이용하여 contextConfigLocation에 있는 root-context들을 불러옵니다.
3. 클라이언트의 요청을 받으면 servlet 태그 안에 있는 설정들이 작동하면서 servlet-context와 root-context를 동시에 같이 불러오며 DispatcherServlet 클래스를 실행시킵니다.

**web.xml**

* servlet 태그 안에 DispatcherServlet과 함께 사용하면 servlet context의 설정 파일임을 명시한다.

```xml
<servlet>
    <servlet-name>appServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
```

* servlet context 설정 파일을 DispatcherServlet의 contextConfigLocation으로 지정하고 있다.

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/spring/root-context.xml</param-value>
</context-param>

<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

* root context 설정 파일을 ContextLoaderListener의 contextConfigLocation으로 지정하고 있다.

**servlet-context.xml**

* servlet-context에 등록되는 빈들은 해당 컨테스트에서만 사용할 수 있습니다.
* **Controller, HandlerMaping, ViewResolver** 와 같은 웹과 연관되어 있는 bean들을 정의합니다.

**root-context.xml**

* root-context에서 등록되는 빈들은 모든 컨텍스트에서 사용할 수 있습니다. (공유 가능)
* **Repository, Service** 와 같은 여러 요청에 대해서 공유해야 하는 bean 들을 정의합니다.
* **servlet-context**내 빈들은 이용이 불가능합니다.

> root-context는 공유가 가능한 반면 servlet-context는 해당 컨텍스트 내에 있는 빈만 사용이 가능하다.
>
> servlet-context 는 컨트롤러 담당, root-context는 서비스와 레포지토리를 담당한다.

**Controller, RestController 차이**

* 회원가입 예제를 구현하던 중 Controller을 통해 view 리턴하는 것이 일반적이었으나, Json 형태로 객체 데이터를 반환할 때는 무조건적으로 RestController를 사용하는 것? 이었나... 헷갈려서 개념을 정리하고자 합니다.
* **@RestController 란???**
    * @RestController는 **@Controller에 @ResponseBody**가 추가된 것입니다. 
    * @Controller는 Model 객체를 만들고 결과에 맞는 뷰 페이지를 리턴합니다.
    * @ResponseBody는 HTTP Response Body에 데이터를 담아 리턴합니다.
    * 주용도는 Json 형태로 객체 데이터를 반환할 때 사용합니다.
    * 데이터를 응답으로 제공하는 REST API를 개발할 때 주로 사용하며 일반적으로 객체를 ResponseEntity로 감싸서 반환할 수도 있습니다.
    * 객체를 반환하기 위해서는 view resolver 대신에 HttpMessageConverter가 동작합니다.
    * 즉, spring4이전에는 아래와 같이 @Controller와 @ResponseBody 어노테이션을 같이 사용했다면 이후 버전은 @RestController로 Restful하게 사용합니다.

```java
@Controller
@ResponseBody
public class MVCController{
    ...
}

@RestController
public class RestFulController{
    ...
}
```
