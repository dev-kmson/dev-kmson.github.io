---
sort: 1
---

# DispatcherServlet

---

    - 프론트 컨트롤러의 역할을 함

    - 스프링에 의해 자동으로 등록되는 서블릿으로써 모든 경로에 대하여 매핑 함

    - DispatcherServlet.doDispatch() 호출

        서블릿 호출 시 HttpServlet 인터페이스의 service()가 호출되게 되어있음
        DispatcherServlet의 부모인 FrameworkServlet에서 service()를 오버라이드 하였으므로
        FrameworkServlet의 service()가 호출되고 이후 DispatcherServlet의 doDispatch()가 호출 됨

        doDispatch()에서 아래의 동작 순서에 해당하는 핵심 기능을 수행 함

    - 동작 순서 

        핸들러 조회 -> 핸들러 어댑터 조회 -> 핸들러 어댑터 실행 -> 핸들러 실행
            -> ModelAndView 반환 -> ViewResolver 호출 -> View 반환 -> 뷰 렌더링

    - 주요 인터페이스 목록
    
        핸들러 매핑: org.springframework.web.servlet.HandlerMapping 
        핸들러 어댑터: org.springframework.web.servlet.HandlerAdapter 
        뷰 리졸버: org.springframework.web.servlet.ViewResolver
        뷰 : org.springframework.web.servlet.View 

## HandlerMapping

    - 핸들러를 찾기 위해 스프링에서 자동으로 등록하는 핸들러 매핑 존재

        RequestMappingHandlerMapping, BeanNameUrlHandlerMapping...

            @RequestMapping으로 매핑된 URL을 찾을 수 있는 
            RequestMappingHandlerMapping 핸들러 매핑으로 가장 먼저 탐색 함        
            이 후, 스프링에 의해 정해진 우선순위에 따라 탐색

    - 핸들러 조회

        URL 호출 시 스프링에 준비된 여러 핸들러 매핑 중에서 호출된 URL과 매핑된 핸들러가 있는지 조회 함
        조회 성공 시 매핑된 핸들러 구현체를 반환하고 없으면 404 status code를 클라이언트에 리턴 함

```java

mappedHandler = getHandler(processedRequest); 
    if (mappedHandler == null) {
        noHandlerFound(processedRequest, response);
        return; 
    }

```

## HandlerAdapter

    - 핸들러 매핑을 통해서 찾은 핸들러를 실행시키기 위해 스프링에서 자동으로 등록하는 핸들러 어댑터 존재

        RequestMappingHandlerAdapter, HttpRequestHandlerAdapter, SimpleControllerHandlerAdapter...

            @RequestMapping에 의해 매핑된 핸들러를 처리할 수 있는 
            RequestMappingHandlerAdapter 핸들러 어댑터로 가장 먼저 탐색 함
            이 후, 스프링에 의해 정해진 우선순위에 따라 탐색

    - 어댑터 조회

        핸들러 매핑이 반환한 구현체의 타입을 처리할 수 있는 어댑터가 있는지 조회 함

```java

HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

```
        
    - 어댑터 실행
    
        어댑터 조회 성공 시 해당 어댑터로 핸들러 매핑이 반환한 구현체를 실행시킴

```java

mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

```

## ViewResolver

    - 논리 이름을 물리 이름으로 변환하기 위해 스프링에서 자동으로 등록하는 뷰 리졸버 존재

        BeanNameViewResolver, InternalResourceViewResolver...

        빈 이름으로 뷰를 찾아서 반환하는 BeanNameViewResolver 뷰 리졸버로 가장 먼저 탐색 함
        이 후, 스프링에 의해 정해진 우선순위에 따라 탐색

    - 뷰 리졸버 호출

        어댑터가 반환한 ModelAndView에서 뷰의 논리 이름을 얻어낸 후
        실제 해당 뷰의 물리 경로로 변환하여 View로 반환

```java

View view;
String viewName = mv.getViewName();
view = resolveViewName(viewName, mv.getModelInternal(), locale, request);


```