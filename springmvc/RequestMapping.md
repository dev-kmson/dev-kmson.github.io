---
sort: 2
---

# RequestMapping

---

## @ RequestMapping

    - 스프링에서 지원하는 어노테이션 기반의 컨트롤러에 사용

    - 클래스 또는 메서드 레벨에서 사용하여 URL을 매핑할 수 있음

    - 어노테이션 기반의 컨트롤러를 지원하는 RequestMappingHandlerMapping, RequestMappingHandlerAdapter

        스프링에서 가장 높은 우선순위를 가진 핸들러 매핑과 핸들러 어댑터임

    - 기존 서블릿과 달리 @RequestMapping은 메서드 레벨에서 URL을 매핑할 수 있으므로 컨트롤러를 유연하게 통합하여 관리할 수 있음

    - 클래스 레벨의 URL과 메서드 레벨의 URL을 조합하여 사용할 수 있음

```java

        @Controller
        @RequestMapping("/springmvc/v3/members")
        public class SpringMemberControllerV3 {
            private MemberRepository memberRepository = MemberRepository.getInstance();

            @GetMapping("/new-form")
            public String newForm() {
                return "new-form";
            }
    
        }

```

        클래스 레벨의 매핑 URL "/springmvc/v3/members와 메서드 레벨의 매핑 URL "/new-form"이 조합되어
        "/springmvc/v3/members/new-form"로 매핑할 수 있음

    - @GetMapping, @PostMapping, @PutMapping 등 HTTP Method를 구분하기 위한 어노테이션도 준비되어 있음

        해당 어노테이션의 내부에 @RequestMapping 어노테이션이 존재 함을 확인할 수 있음

```java

        @RequestMapping(method = RequestMethod.GET)
        public @interface GetMapping {
            ...
        }

```

        기존에는 @RequestMapping의 method 옵션을 이용하여 HTTP Method를 구분하였으나
        스프링에서 어노테이션을 이용하여 구분할 수 있도록 지원 함 

    - 메서드의 매개변수로 @RequestParam을 사용하여 request.getParameter("...")을 사용하는 것과 같은 효과를 얻을 수 있음

```java

        @PostMapping("/save")
        public String save(@RequestParam("username") String username, @RequestParam("age") int age, Model model){
                    ...
        }

```

## RequestMappingHandlerMapping

    - @RequestMapping을 지원하는 핸들러 매핑

    - 스프링 빈 중에서 @RequestMapping 또는 @Controller가 클래스 레벨에 작성되어 있을 경우 핸들러가 취급할 수 있는 매핑 정보로 인식 함

## 다양한 매핑 요청 방법

```java

// @RestController가 붙으면 반환 값으로 뷰를 찾지 않고 HTTP 메시지 바디에 바로 결과 값을 입력 함
@RestController
  public class MappingController {

    private Logger log = LoggerFactory.getLogger(getClass());

    /**
     * 기본 요청
     * 둘다 허용 /hello-basic, /hello-basic/
     * HTTP 메서드 모두 허용 GET, HEAD, POST, PUT, PATCH, DELETE */
    @RequestMapping("/hello-basic")
    public String helloBasic() {
        log.info("helloBasic");
        return "ok";
    }

    /**
     * method 특정 HTTP 메서드 요청만 허용
     * GET, HEAD, POST, PUT, PATCH, DELETE
     */
    @RequestMapping(value = "/mapping-get-v1", method = RequestMethod.GET)
    public String mappingGetV1() {
        log.info("mappingGetV1");
        return "ok";
    }

    /**
     * 편리한 축약 애노테이션 (코드보기) * @GetMapping
     * @PostMapping
     * @PutMapping
     * @DeleteMapping
     * @PatchMapping
     */
    @GetMapping(value = "/mapping-get-v2")
    public String mappingGetV2() {
        log.info("mapping-get-v2");
        return "ok";
    }

    /**
     * PathVariable 사용
     * 변수명이 같으면 생략 가능
     * @PathVariable("userId") String userId -> @PathVariable userId */
    @GetMapping("/mapping/{userId}")
    public String mappingPath(@PathVariable("userId") String data) {
        log.info("mappingPath userId={}", data);
        return "ok";
    }

    /**
     * PathVariable 사용 다중
     */
    @GetMapping("/mapping/users/{userId}/orders/{orderId}")
    public String mappingPath(@PathVariable String userId, @PathVariable Long
            orderId) {
        log.info("mappingPath userId={}, orderId={}", userId, orderId);
        return "ok";
    }

    /**
     * 파라미터로 추가 매핑 (특정 쿼리 파라미터의 조건 여부)
     * params="mode",
     * params="!mode"
     * params="mode=debug"
     * params="mode!=debug"
     * params = {"mode=debug","data=good"}
     */
    @GetMapping(value = "/mapping-param", params = "mode=debug")
    public String mappingParam() {
        log.info("mappingParam");
        return "ok";
    }

    /**
     *특정 헤더로 추가 매핑 (특정 헤더의 조건 여부)
     * headers="mode",
     * headers="!mode"
     * headers="mode=debug"
     * headers="mode!=debug"
     */
    @GetMapping(value = "/mapping-header", headers = "mode=debug")
    public String mappingHeader() {
        log.info("mappingHeader");
        return "ok";
    }

    /**
     * Content-Type 헤더 기반 추가 매핑 Media Type * consumes="application/json"
     * consumes="!application/json"
     * consumes="application/*"
     * consumes="*\/*"
     * MediaType.APPLICATION_JSON_VALUE
     */
    @PostMapping(value = "/mapping-consume", consumes = "application/json")
    public String mappingConsumes() {
        log.info("mappingConsumes");
        return "ok";
    }

    /**
     * Accept 헤더 기반 Media Type * produces = "text/html"
     * produces = "!text/html" * produces = "text/*"
     * produces = "*\/*"
     */
    @PostMapping(value = "/mapping-produce", produces = "text/html")
    public String mappingProduces() {
        log.info("mappingProduces");
        return "ok";
    }

    /**
     * 다양한 header 정보를 메서드의 매개변수로 받을 수 있음
     */
    @RequestMapping("/headers")
    public String headers(HttpServletRequest request,
                          HttpServletResponse response,
                          HttpMethod httpMethod,
                          Locale locale,
                          @RequestHeader MultiValueMap<String, String> headerMap,
                          @RequestHeader("host") String host,
                          @CookieValue(value = "myCookie", required = false) String cookie
    ) {
        
        return ok;
        
    }
    
}

```
매핑 관련 공식 메뉴얼  
[요청 파라미터 목록 공식 메뉴얼](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-arguments)  
[응답 목록 공식 메뉴얼](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-return-types)

