---
sort: 9
---

# ErrorPage

---

## Servlet ErrorPage

```java

@Component
  public class WebServerCustomizer implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {
    
      @Override
      public void customize(ConfigurableWebServerFactory factory) {
          
          ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND, "/error-page/404");
          ErrorPage errorPage500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-page/500");
          ErrorPage errorPageEx = new ErrorPage(RuntimeException.class, "/error-page/500");
          
          factory.addErrorPages(errorPage404, errorPage500, errorPageEx);
          
      }
}


```

```java

@Slf4j
  @Controller
  public class ErrorPageController {
    
      @RequestMapping("/error-page/404")
      public String errorPage404(HttpServletRequest request, HttpServletResponse response) {
          log.info("errorPage 404");
          return "error-page/404";
      }
      
      @RequestMapping("/error-page/500")
      public String errorPage500(HttpServletRequest request, HttpServletResponse response) {
          log.info("errorPage 500");
          return "error-page/500";
      }
}

```

    서블릿이 제공하는 오류 페이지 기능을 사용하면 WebServerCustomizer를 통하여
    HttpStatus 또는 exception 종류 별로 일일이 오류 페이지를 등록하여야 하고
    오류 페이지 관련 컨트롤러를 만들어 매핑시켜줘야 하는 불편함이 존재함

## Spring ErrorPage

    서블릿의 오류 페이지 처리와는 다르게 WebServerCustomizer, 오류 처리 컨트롤러를 별도로 만들지 않아도
    스프링의 ErrorMvcAutoConfiguration 클래스가 오류 페이지를 자동으로 등록하는 역할을 하고
    스프링이 자동으로 생성한 BasicErrorController가 오류 페이지를 매핑함

    개발자는 application.properties에 설정한 server.error.path의 경로에 따라
    오류 페이지를 만들어두기만 하면 됨 (server.error.path는 /error 경로가 디폴트로 잡혀있음)