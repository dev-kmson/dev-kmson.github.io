---
sort: 6
---

# Interceptor

---

## Filter

    - 웹과 관련된 공통 관심사(cross-cutting concern)를 처리하기 위해 서블릿이 제공하는 기능

        Filter 적용 시 HTTP 요청에 의해 WAS가 서블릿을 호출하게 되면 
        Filter에 의해 공통 관심사에 대한 처리 후에 서블릿이 호출됨

        HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러

### 필터 인터페이스 

```java

    public interface Filter {
        
        public default void init(FilterConfig filterConfig) throws ServletException {}
    
        public void doFilter(ServletRequest request, ServletResponse response,
                             FilterChain chain) throws IOException, ServletException;
    
        public default void destroy() {}
        
    }

```

        init(): 필터 초기화 메서드, 서블릿 컨테이너가 생성될 때 호출됨
        doFilter(): 고객의 요청이 올 때 마다 해당 메서드가 호출됨 
        destroy(): 필터 종료 메서드, 서블릿 컨테이너가 종료될 때 호출됨

### 필터 구현

```java
    
    @Slf4j
    public class LogFilter implements Filter {
        
        // init(), doFilter(), destroy() 오버라이딩을 통한 기능 구현
    
        @Override
        public void doFilter(ServletRequest request, ServletResponse response,
                             FilterChain chain) throws IOException, ServletException {
    
            // ...
            
            try {
                // ...
                chain.doFilter(request, response);
            } catch (Exception e) {
                throw e;
            } finally {
                // ...
            }
            
        }
        
    }

```
        Servlet에서 제공하는 Filter 기능은 HTTP만을 위한 기능이 아니므로 
        ServletRequest, ServletResponse를 매개변수로 받음
        HTTP에 특화된 기능을 사용하기 위해선 HttpServletRequest, HttpServletResponse로 
        다운캐스팅하여 사용하여야 함
    
        필터는 체인으로 구성되어 있어 공통 관심사에 대한 처리 후에는 
        반드시 chain.doFilter(request, response)를 호출하여
        다음 필터가 존재하면 필터를 호출, 없으면 서블릿을 호출하도록 하여야 함
        참고로, spring의 Interceptor와는 다르게 
        다음 필터 또는 서블릿을 호출하면서 request, response를
        다른 객체로 변경하여 호출할 수 있음 
        
### 필터 등록

```java

    @Configuration
        public class WebConfig {
        
        @Bean
        public FilterRegistrationBean logFilter() {
            
            FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
            
            filterRegistrationBean.setFilter(new LogFilter());
            filterRegistrationBean.setOrder(1);
            filterRegistrationBean.addUrlPatterns("/*");
            
            return filterRegistrationBean;
        }
        
    }

```
    
        필터를 등록하는 여러 방법들 중 스프링 부트를 사용하는 경우에는 FilterRegistrationBean을 이용함
        
        @SpringBootApplication이 붙은 클래스에 @ServletComponentScan를 추가하고
        Filter를 구현한 클래스에 @WebFilter를 추가하여 필터를 등록하는 방법도 존재하지만
        Filter에 대한 순번을 부여할 수 없어 FilterRegistrationBean을 주로 이용함

## Interceptor

    - 웹과 관련된 공통 관심사(cross-cutting concern)를 처리하기 위해 스프링 MVC가 제공하는 기능

        Interceptor 적용 시 HTTP 요청에 의해 WAS가 서블릿을 호출하게 되면 
        Dispatcher Servlet이 스프링 인터셉터를 호출하여 
        공통 관심사에 대한 처리 후에 컨트롤러를 호출함

        HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러

### 인터셉터 인터페이스

```java

    public interface HandlerInterceptor {
    
        default boolean preHandle(HttpServletRequest request,
            HttpServletResponse response, Object handler) throws Exception {}
    
        default void postHandle(HttpServletRequest request, HttpServletResponse response,
            Object handler, @Nullable ModelAndView modelAndView) throws Exception {}
    
        default void afterCompletion(HttpServletRequest request, HttpServletResponse response,
            Object handler, @Nullable Exception ex) throws Exception {}
    }

```

        preHandle -> 

            DispatcherServlet이 HandlerAdapter를 호출하기 이전에 preHandle()를 호출함

            preHandle의 결과 값이 false인 경우 이후 존재하는 인터셉터, 핸들러 어뎁터를 호출하지 않음

        postHandle -> 

            DispatcherServlet이 HandlerAdapter 호출 이후에 postHandle()를 호출함

            modelAndView 정보를 받아 이용할 수 있음

            컨트롤러에서 예외 발생 시 호출되지 않음

        afterCompletion -> 

            View가 rendering된 이후에 호출됨

            예외가 발생하는 경우에도 무조건 호출되므로 예외와 무관하게 공통 관심사를 처리해야하는 경우
            afterCompletion()을 사용해야하고 exception 정보를 받아 이용할 수 있음 

        Filter와 달리 handler에 대한 정보를 preHandle, postHandle, afterCompletion 모두 매개변수로 받아 이용할 수 있음

### 인터셉터 구현

```java

    @Slf4j
    public class LogInterceptor implements HandlerInterceptor {
    
        // preHandle(), postHandle(), afterCompletion() 오버라이딩을 통한 기능 구현
    
    }

```

        Filter와 달리 같은 체인 구성으로 되어 있음에도 chain.doFilter(request, response)와 같이
        다음 인터셉터 또는 핸들러 어뎁터 호출을 위한 메서드를 개발자가 호출하지 않아도 됨

### 인터셉터 등록

```java
    
    @Configuration
      public class WebConfig implements WebMvcConfigurer {
        
          @Override
          public void addInterceptors(InterceptorRegistry registry) {
              
              registry.addInterceptor(new LogInterceptor())
                      .order(1)
                      .addPathPatterns("/**")
                      .excludePathPatterns("/css/**", "/*.ico", "/error");
              
         }
        //...
    }

```

        인터셉터 등록을 위해 WebMvcConfigurer를 구현하여야 함

        WebMvcConfigurer가 제공하는 addInterceptors 메서드를 이용하여 인터셉터 등록

## 참고

    서블릿이 제공하는 필터보다 스프링이 제공하는 인터셉터가 더 정교하고 다양한 기능을 지원하므로 필터보단 스프링의 인터셉터의 이용이 권장됨
 
[스프링 URL 패턴](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/util/pattern/PathPattern.html)
