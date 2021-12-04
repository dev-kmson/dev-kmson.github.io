---
sort: 8
---

# ArgumentResolver 활용

---

```java

//@GetMapping("/")
public String homeLoginV3Spring(
        @SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Member loginMember, 
        Model model) {

    if (loginMember == null) {
        return "home";
    }

    model.addAttribute("member", loginMember);
    
    return "loginHome";

}

```

```java

@GetMapping("/")
public String homeLoginV3ArgumentResolver(@Login Member loginMember, Model model) {

    if (loginMember == null) {
        return "home";
    }
    
    model.addAttribute("member", loginMember);
        
    return "loginHome";
        
}

```

    세션과 같이 지속적으로 개발자가 이용하게 되는 로직의 경우 ArgumentResolver를 활용하면
    어노테이션 하나로 처리할 수 있음

## Annotation 등록

```java

@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface Login {
}

```

## ArgumentResolver 구현

```java

@Slf4j
public class LoginMemberArgumentResolver implements HandlerMethodArgumentResolver {

    @Override
    public boolean supportsParameter(MethodParameter parameter) { 
        
        log.info("supportsParameter 실행");
        
        boolean hasLoginAnnotation = parameter.hasParameterAnnotation(Login.class);
        boolean hasMemberType = Member.class.isAssignableFrom(parameter.getParameterType());
        
        return hasLoginAnnotation && hasMemberType;
        
    }
    
    @Override
    public Object resolveArgument(MethodParameter parameter,
                                  ModelAndViewContainer mavContainer, NativeWebRequest webRequest,
                                  WebDataBinderFactory binderFactory) throws Exception {
    
        log.info("resolveArgument 실행");
        
        HttpServletRequest request = (HttpServletRequest) webRequest.getNativeRequest();
        
        HttpSession session = request.getSession(false);
        if (session == null) {
            return null;
        }
        
        return session.getAttribute(SessionConst.LOGIN_MEMBER);
        
    }
    
}

```

## ArgumentResolver 등록

```java

@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
          resolvers.add(new LoginMemberArgumentResolver());
      }
    
    //...
    
}

```

    로그인 세션 처리를 위한 새로운 ArgumentResolver를 등록함으로써
    @SessionAttribute를 사용하여 동일한 속성 값을 넣어주어야 하는 불편함을 해소하고
    간편하게 @Login만으로 로그인 세션 처리를 해결하였음

    이와 같이, ArgumentResolver를 활용하면 공통적으로 사용되는 로직을 어노테이션으로
    처리함으로써 개발자의 편의성을 증대할 수 있음