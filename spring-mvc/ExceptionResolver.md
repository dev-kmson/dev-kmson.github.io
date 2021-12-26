---
sort: 10
---

# ExceptionResolver

    HandlerExceptionResolver를 말하며 인터페이스로써
    구현체로는 ExceptionHandlerExceptionResolver, ResponseStatusExceptionResolver,
    DefaultHandlerExceptionResolver가 있음

```java

    public interface HandlerExceptionResolver {
        ModelAndView resolveException(HttpServletRequest request, 
                HttpServletResponse response, Object handler, Exception ex);
    }

```

---

## 예외 처리 흐름

    WAS -> DisPatcherServlet -> 
        [ 
            preHandle(Interceptor) -> HandlerAdapter -> Handler** 예외발생 -> 
            ExceptionResolver 예외 해결 시도** -> afterCompletion(Interceptor) 
        ]
    -> WAS** 예외 확인(스프링이 등록한 예외 매핑 URL 호출) -> BasicErrorController(예외 처리) -> WAS

    핸들러(컨트롤러)에서 예외 발생 시 WAS로 예외가 전파되기 전에 ExceptionResolver가 해당 예외를 해결하기 위해 시도함
    여러 ExceptionResolver(ExceptionHandlerExceptionResolver, ResponseStatusExceptionResolver,
    DefaultHandlerExceptionResolver)가 발생한 예외를 해결하지 못하면 발생했던 예외가 그대로 WAS로 전파됨
    WAS는 예외를 확인 후에 예외 오류 처리 페이지 URL을 호출(스프링이 등록한 예외 매핑 URL) 함
    오류 처리 페이지 URL(예외 매핑 URL)은 BasicErrorController에 등록되어 있으므로 해당 컨트롤러가 호출되며
    Accept 헤더에 따라 html 또는 json 형식으로 응답하여 다시 WAS로 전달함

    ExceptionResolver가 Handler에서 발생한 예외를 정상적으로 해결하였다면
    개발자가 의도한 정상 응답 혹은 스프링이 자동 처리한 정상 응답 결과가 WAS로 전달됨

## HandlerExceptionResolver 직접 구현

```java

    @Slf4j
    public class MyHandlerExceptionResolver implements HandlerExceptionResolver {
    
        @Override
        public ModelAndView resolveException(HttpServletRequest request,
            HttpServletResponse response, Object handler, Exception ex) {
            
            try {
                
                if (ex instanceof IllegalArgumentException) {
                    log.info("IllegalArgumentException resolver to 400");
                    response.sendError(HttpServletResponse.SC_BAD_REQUEST, ex.getMessage());
                    return new ModelAndView();
                }
                
            } catch (IOException e) {
                  log.error("resolver ex", e);
            }
            return null;
        }
    }

```
 
```java

    @Slf4j
    public class UserHandlerExceptionResolver implements HandlerExceptionResolver {
    
        private final ObjectMapper objectMapper = new ObjectMapper();
        
        @Override
        public ModelAndView resolveException(HttpServletRequest request,
                HttpServletResponse response, Object handler, Exception ex) {
            
            try {
                
                if (ex instanceof UserException) {
                    
                    log.info("UserException resolver to 400");
    
                    response.setStatus(HttpServletResponse.SC_BAD_REQUEST);
                    
                    String acceptHeader = request.getHeader("accept");
                    if ("application/json".equals(acceptHeader)) {
                        
                        Map<String, Object> errorResult = new HashMap<>();
                        errorResult.put("ex", ex.getClass());
                        errorResult.put("message", ex.getMessage());
                        String result = objectMapper.writeValueAsString(errorResult);
                        
                        response.setContentType("application/json");
                        response.setCharacterEncoding("utf-8");
                        response.getWriter().write(result);
                        return new ModelAndView();
                        
                    } else {
                        //TEXT/HTML
                        return new ModelAndView("error/500");
                    }
                }
                
            } catch (IOException e) {
                log.error("resolver ex", e);
            }
            
            return null;
        }
        
    }

```

```java

    @Configuration
    public class WebConfig implements WebMvcConfigurer {
    
        @Override
        public void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
            resolvers.add(new MyHandlerExceptionResolver());
            resolvers.add(new UserHandlerExceptionResolver());
        }
        
    }

```

    HandlerExceptionResolver 인터페이스를 구현하여 WebMvcConfigurer를 통해 등록시킴
    HandlerExceptionResolver를 직접 구현하기에는 예외 타입마다 처리하는 방법을
    작성하여야 하며 해당 인터페이스의 반환 타입이 ModelAndView 타입이기 때문에
    API(application/json) 요청인 경우 json 형식으로 응답하기 위해서는
    HttpServletResponse에 일일이 헤더와 바디를 구성하여야 하는 등 복잡한 과정을 거치게 됨

    Spring에서는 이와같이 복잡한 과정을 대신 처리해주는 여러 ExceptionResolver를 구현해놓음
    
    ExceptionResolver의 반환 유형에 따른 의미

        빈 ModelAndView : 
            new ModelAndView()처럼 빈 ModelAndView를 반환하면 뷰를 렌더링 하지 않고 정상 흐름으로 서블릿이 리턴됨
        ModelAndView 지정 : 
            ModelAndView에 View, Model 등의 정보를 지정해서 반환하면 뷰를 렌더링함
        null :
            null을 반환하면 다음 ExceptionResolver를 찾아서 실행하며, 만약 처리할 수 있는
            ExceptionResolver가 더 이상 존재하지 않으면 기존에 발생한 예외를 서블릿 밖으로 넘김

## ResponseStatusExceptionResolver

    HTTP 상태 코드를 지정해줌
    
    예외가 발생했을 때 클라이언트에 내부 오류(5xx)가 아닌 다른 
    오류 코드로 변경하여 응답해야 하는 경우에 사용

```java

    // reason에 기입되는 메시지는 MessageSource(messages.properties)에서 찾는 기능도 제공
    @ResponseStatus(code = HttpStatus.BAD_REQUEST, reason = "잘못된 요청 오류") 
    public class BadRequestException extends RuntimeException {
        //...
    }

```

```java

    @GetMapping("/api/response-status-ex1")
    public String responseStatusEx1() {
        throw new BadRequestException();
    }

```

    사용자가 정의한 예외 클래스에 @ResponseStatus를 작성해두면
    해당 유형의 예외 발생 시에 ResponseStatusExceptionResolver가
    @ResponseStatus가 붙어있는지 확인 후에 HTTP 상태 코드를 변경함

    단, 사용자가 정의한 예외 클래스가 아닌 일반적인 예외 클래스인 경우에는
    @ResponseStatus를 작성할 방법이 없으며 애노테이션을 사용하기 때문에
    조건에 따라 동적으로 HTTP 상태 코드를 변경하는 것도 어려움

```java

    @GetMapping("/api/response-status-ex2")
    public String responseStatusEx2() {
        try {
            //...
        } catch (IllegalArgumentException e) {
            throw new ResponseStatusException(HttpStatus.NOT_FOUND,
                "error.bad", new IllegalArgumentException());   
        }
    }

```

    @ResponseStatus 대신 ResponseStatusException 예외를 사용하면
    조건에 따라 동적으로 HTTP 상태를 변경할 수 있으며 특정 예외가 발생하였을 때
    catch문에서 ResonseStatusException 예외를 던질 수 있기 때문에
    사용자가 정의한 예외 클래스가 아니더라도 HTTP 상태 코드를 용이하게 변경할 수 있음

## DefaultHandlerExceptionResolver

    스프링 내부에서 발생하는 스프링 예외를 해결하는데 사용됨
    대표적인 예로 RequestMapping의 파라미터 바인딩 시에 타입이 맞지 않으면 
    내부에서 TypeMismatchException이 발생하며, 이 경우에 
    DefaultHandlerExceptionResolver가 오류 해결을 시도함

        타입 불일치는 일반적으로 클라이언트에서 잘못된 정보를 넘긴 경우가 대부분이므로
        HTTP 상태 코드를 500 오류가 아닌 400 오류로 변경함

## ExceptionHandlerExceptionResolver

    브라우저를 통한 클라이언트 요청인 경우 BasicErrorController를 통하여
    에러 페이지를 표출해주면 그만이지만 API 요청인 경우에는 단순 에러 페이지
    표출이 아닌 정확하고 세밀한 데이터 제어가 필요하며 연계되는 API 규격 별로
    똑같은 예외인 경우에도 다른 예외 응답을 내려주어야 하는 경우가 발생함
    또한, 같은 예외더라도 특정 컨트롤러 별로 다른 예외 응답을 내려주어야 
    하는 경우도 발생함
    
    이와 같은 복잡한 문제를 해결하기 위해 ExceptionHandlerExceptionResolver를 사용함

```java

    @Data
    @AllArgsConstructor
    public class ErrorResult {
        private String code;
        private String message;
    }

```

```java

    @Slf4j
    @RestController
    public class ApiExceptionV2Controller {
    
        @ResponseStatus(HttpStatus.BAD_REQUEST)
        @ExceptionHandler(IllegalArgumentException.class)
        public ErrorResult illegalExHandle(IllegalArgumentException e) {
            log.error("[exceptionHandle] ex", e);
            return new ErrorResult("BAD", e.getMessage());
        }
    
        @ExceptionHandler
        public ResponseEntity<ErrorResult> userExHandle(UserException e) {
            log.error("[exceptionHandle] ex", e);
            ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
            return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
        }
    
        @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
        @ExceptionHandler
        public ErrorResult exHandle(Exception e) {
            log.error("[exceptionHandle] ex", e);
            return new ErrorResult("EX", "내부 오류");
        }
    
        @GetMapping("/api2/members/{id}")
        public MemberDto getMember(@PathVariable("id") String id) {
    
            if (id.equals("ex")) {
                throw new RuntimeException("잘못된 사용자");
            }
            if (id.equals("bad")) {
                throw new IllegalArgumentException("잘못된 입력 값");
            }
            if (id.equals("user-ex")) {
                throw new UserException("사용자 오류");
            }
    
            return new MemberDto(id, "hello " + id);
        }
    
        @Data
        @AllArgsConstructor
        static class MemberDto {
            private String memberId;
            private String name;
        }
        
    }

```

    @ExceptionHandler를 통하여 해당하는 컨트롤러 내에서만 특정 예외 타입을 지정하여
    예외 발생 시 @ExceptionHandler가 작성된 메서드가 실행되어 예외를 처리함

    지정한 예외 타입 또는 그 하위 예외 타입을 모두 처리할 수 있으며
    하위 클래스일수록 더 높은 우선순위를 가짐

    예외 타입 지정은 여러개가 가능하며 메서드의 파라미터로 넘어온 예외 타입과
    @ExceptionHandler 내의 작성한 예외 타입이 일치하는 경우에는
    @ExceptionHandler 내의 예외 타입 작성을 생략할 수 있음

    컨트롤러에서 예외 발생 시 해당 예외가 컨트롤러 밖으로 던져지며
    예외가 발생하였으므로 ExceptionResolver가 작동함
    가장 우선순위가 높은 ExceptionHandlerExceptionResolver가 가장
    먼저 실행되어 예외가 발생한 컨트롤러에 해당 예외 타입을 처리할 수 있는
    @ExceptionHandler가 존재하는지 확인함
    존재한다면, @ExceptionHandler가 작성되어 있는 메서드를 실행하고
    정상 응답으로 처리되어 WAS에 전달됨

    반환타입으로 ResponseEntity를 사용하여 HTTP 응답 코드를 동적으로
    변경할 수 있으며 @ResponseStatus가 같이 사용되고 있긴 하나
    @ResponseStatus는 애노테이션으로써 조건에 따라 동적으로 HTTP 응답
    코드를 변경할 수 없음

    반환타입으로 ModelAndView를 사용하여 오류 화면을 응답하는데 
    사용하는 것 또한 가능하나 일반적으로 브라우저 요청에서의 오류 응답에는
    ExceptionHandlerExceptionResolver를 사용하지는 않음

[@ExceptionHandler 파라미터, 반환타입](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-exceptionhandler-args)

```java

    @Slf4j
    @RestControllerAdvice
    public class ExControllerAdvice {
         
        @ResponseStatus(HttpStatus.BAD_REQUEST)
        @ExceptionHandler(IllegalArgumentException.class)
        public ErrorResult illegalExHandle(IllegalArgumentException e) {
            log.error("[exceptionHandle] ex", e);
            return new ErrorResult("BAD", e.getMessage());
        }
    
        @ExceptionHandler
        public ResponseEntity<ErrorResult> userExHandle(UserException e) {
            log.error("[exceptionHandle] ex", e);
            ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
            return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
        }
    
        @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
        @ExceptionHandler
        public ErrorResult exHandle(Exception e) {
            log.error("[exceptionHandle] ex", e);
            return new ErrorResult("EX", "내부 오류");
        }
    }

```

    컨트롤러 별로 따로 적용되어야 할 예외 타입이 있다면 컨트롤러 별로 작성되어야 하지만
    공통적으로 처리되어도 상관없는 예외 타입의 경우에는 각 컨트롤러마다 중복된 코드들이
    계속 작성되고 오류 처리에 대한 코드가 컨트롤러에 같이 작성되어 있음

    이를 해결하기 위해 @ControllerAdvice 또는 @RestControllerAdvice를 사용하여
    모든 컨트롤러에서 공통적으로 특정 예외 타입에 처리되도록 하거나 특정 컨트롤러 대상을
    지정하여 적용할 수도 있음

```java

    // 애노테이션 지정 방식
    @ControllerAdvice(annotations = RestController.class)
    public class ExampleAdvice1 {}
    
    // 패키지 지정 방식 (패키지 하위의 모든 컨트롤러 해당)
    @ControllerAdvice("org.example.controllers")
    public class ExampleAdvice2 {}
    
    // 클래스 지정 방식
    @ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
    public class ExampleAdvice3 {}

```

    대상 컨트롤러를 지정하지않으면 모든 컨트롤러에 적용됨

[@ControllerAdvice 대상 지정](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-controller-advice)
