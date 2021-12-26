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
    
    @RequestMapping(value = "/error-page/500", produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<Map<String, Object>> errorPage500Api(
            HttpServletRequest request, HttpServletResponse response) {
        
        log.info("API errorPage 500");

        Exception ex = (Exception) request.getAttribute(ERROR_EXCEPTION);
        
        Map<String, Object> result = new HashMap<>();
        
        result.put("status", request.getAttribute(ERROR_STATUS_CODE));
        result.put("message", ex.getMessage());
        
        Integer statusCode = (Integer) request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);
        return new ResponseEntity(result, HttpStatus.valueOf(statusCode));
    }
}

```

    서블릿이 제공하는 오류 페이지 기능을 사용하면 WebServerCustomizer를 통하여
    HttpStatus 또는 exception 종류 별로 일일이 오류 페이지를 등록하여야 하고
    오류 페이지 관련 컨트롤러를 만들어 매핑시켜줘야 하는 불편함이 존재함

    또한, 브라우저 요청인지 API 요청인지에 구분하기 위해 produces 옵션을 설정하여
    API 요청인 경우에는 json 형식으로 응답할 수 있도록 작성하여야 함

## Spring ErrorPage

    서블릿의 오류 페이지 처리와는 다르게 WebServerCustomizer, 오류 처리 컨트롤러를 별도로 만들지 않아도
    스프링의 ErrorMvcAutoConfiguration 클래스가 오류 페이지를 자동으로 등록하는 역할을 하고
    스프링이 자동으로 생성한 BasicErrorController가 오류 페이지를 매핑함

    개발자는 application.properties에 설정한 server.error.path의 경로에 따라
    오류 페이지를 만들어두기만 하면 됨 (server.error.path는 /error 경로가 디폴트로 잡혀있음)

```java
    // BasicErrorController의 errorHtml(), error()로 구분되어 클라이언트의 요청 Accept 헤더에 따라 처리함
    @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
    public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {}
    
    @RequestMapping
    public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {}

```

    또한, BasicErrorController에서 오류를 처리할 때 accept 헤더가 text/html로 온 요청인지
    아닌지를 구분하여 처리하기 때문에 브라우저 요청인지 API 요청인지에 따라 html 응답, json 응답을
    개발자가 일일이 구현하지 않아도 됨

## 예외 처리 흐름

    WAS -> DisPatcherServlet -> 
        [ preHandle(Interceptor) -> HandlerAdapter -> Handler** 예외발생 -> afterCompletion(Interceptor) ] 
        -> WAS** 예외 확인(스프링이 등록한 예외 매핑 URL 호출) -> BasicErrorController(예외 처리) -> WAS

    핸들러(컨트롤러)에서 예외 발생 시 인터셉터의 afterCompletion 호출 후에 WAS까지 예외가 전파 됨
    WAS는 예외를 확인 후에 예외 오류 처리 페이지 URL을 호출(스프링이 등록한 예외 매핑 URL) 함
    오류 처리 페이지 URL(예외 매핑 URL)은 BasicErrorController에 등록되어 있으므로 해당 컨트롤러가 호출되며
    Accept 헤더에 따라 html 또는 json 형식으로 응답하여 다시 WAS로 전달함

    WAS가 오류 처리 페이지 URL을 개발자가 지정하지 않아도 알아서 호출할 수 있는 이유는
    스프링의 ErrorMvcAutoConfiguration 클래스가 오류 페이지를 자동으로 등록하기 때문임

