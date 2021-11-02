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

## 다양한 매핑 요청 방법

    - get방식 쿼리파리미터, post방식 Form전송

```java

        // @RestController가 붙으면 반환 값으로 뷰를 찾지 않고 HTTP 메시지 바디에 바로 결과 값을 입력 함
        @RestController
          public class MappingController {
        
            /**
             * 기본 요청
             * 둘다 허용 /hello-basic, /hello-basic/
             * HTTP 메서드 모두 허용 GET, HEAD, POST, PUT, PATCH, DELETE */
            @RequestMapping("/hello-basic")
            public String helloBasic() {
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
             * PathVariable 사용
             * 변수명이 같으면 생략 가능
             * @PathVariable("userId") String userId -> @PathVariable userId */
            @GetMapping("/mapping/{userId}")
            public String mappingPath(@PathVariable("userId") String data) {
                log.info("mappingPath userId={}", data);
                return "ok";
            }
        
            /**
             * PathVariable 다중 사용
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
             * @RequestParam 사용
             * - 파라미터 이름으로 바인딩
             */
            @ResponseBody
            @RequestMapping("/request-param-v2")
            public String requestParamV2(
                    @RequestParam("username") String memberName,
                    @RequestParam("age") int memberAge) {

                log.info("username={}, age={}", memberName, memberAge);

                return "ok";

            }

            /**
             * @RequestParam 사용
             * HTTP 파라미터 이름이 변수 이름과 같으면 @RequestParam(name="xx") 생략 가능 
             */
            @ResponseBody
            @RequestMapping("/request-param-v3")
            public String requestParamV3(
                    @RequestParam String username,
                    @RequestParam int age) {

                log.info("username={}, age={}", username, age);

                return "ok";

            }

            /**
             * @RequestParam 사용
             * String, int 등의 단순 타입이면 @RequestParam 도 생략 가능 
             * 명시적이지 않아 권장하지 않음
             */
            @ResponseBody
            @RequestMapping("/request-param-v4")
            public String requestParamV4(String username, int age) {

                log.info("username={}, age={}", username, age);

                return "ok";

            }

            /**
             * @RequestParam.required
             * /request-param-required -> username이 없으므로 예외
             * /request-param-required?username= -> 빈문자로 통과
             * /request-param-required?username=xx -> age에 null 대입 불가하여 오류
             * 필수값이 아닌 경우 자동으로 null을 대입 시킴
             * int age -> null을 int에 입력하는 것은 불가능, 따라서 Integer 변경해야 함(또는 다음에 나오는 defaultValue 사용) 
             */
            @ResponseBody
            @RequestMapping("/request-param-required")
            public String requestParamRequired(
                    @RequestParam(required = true) String username,
                    @RequestParam(required = false) Integer age) {

                log.info("username={}, age={}", username, age);

                return "ok";

            }

            /**
             * @RequestParam
             * - defaultValue 사용 *
             * 참고: defaultValue는 빈 문자의 경우에도 적용 
             * /request-param?username= -> username=guest
             */
            @ResponseBody
            @RequestMapping("/request-param-default")
            public String requestParamDefault(
                    @RequestParam(required = true, defaultValue = "guest") String username,
                    @RequestParam(required = false, defaultValue = "-1") int age) {

                log.info("username={}, age={}", username, age);

                return "ok";

            }

            /**
             * @RequestParam Map, MultiValueMap
             * Map(key=value)
             * MultiValueMap(key=[value1, value2, ...] 
             * ex) (key=userIds, value=[id1, id2])
             */
            @ResponseBody
            @RequestMapping("/request-param-map")
            public String requestParamMap(@RequestParam Map<String, Object> paramMap) {

                log.info("username={}, age={}", paramMap.get("username"), paramMap.get("age"));

                return "ok";

            }
            /**
             * @ModelAttribute 사용
             * 요청 파라미터를 객체에 바인딩 함
             * 참고: model.addAttribute(helloData) 코드도 함께 자동 적용됨
             */
            @ResponseBody
            @RequestMapping("/model-attribute-v1")
            public String modelAttributeV1(@ModelAttribute HelloData helloData) {

                log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());

                return "ok";

            }

            /**
             * @ModelAttribute 생략 가능
             * String, int 같은 단순 타입 = @RequestParam
             * argument resolver 로 지정해둔 타입 외 = @ModelAttribute 
             * argument resolver -> 스프링에서 지정해둔 @ModelAttribute로 지정하지 않을 객체들
             */
            @ResponseBody
            @RequestMapping("/model-attribute-v2")
            public String modelAttributeV2(HelloData helloData) {

                log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());

                return "ok";

            }
            
        }

```

    - HTTP 바디

```java

        @RestController 
        public class MappingController {
            /**
             * HttpServlet을 이용하여 Http message body 가져옴
             */
            @PostMapping("/request-body-string-v1")
            public void requestBodyString(HttpServletRequest request, HttpServletResponse response) throws IOException {

                ServletInputStream inputStream = request.getInputStream();
                String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

                log.info("messageBody={}", messageBody);

                response.getWriter().write("ok");

            }

            /**
             * InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회 
             * OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력 
             */
            @PostMapping("/request-body-string-v2")
            public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {

                String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

                log.info("messageBody={}", messageBody);

                responseWriter.write("ok");

            }

            /**
             * HttpEntity: HTTP header, body 정보를 편리하게 조회
             * - 메시지 바디 정보를 직접 조회(@RequestParam X, @ModelAttribute X)
             * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용 
             *
             * 응답에서도 HttpEntity 사용 가능
             * - 메시지 바디 정보 직접 반환(view 조회X)
             * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용 
             **/
            @PostMapping("/request-body-string-v3")
            public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) {

                String messageBody = httpEntity.getBody();

                log.info("messageBody={}", messageBody);

                return new HttpEntity<>("ok");

            }

            /**
             * @RequestBody
             * - 메시지 바디 정보를 직접 조회(@RequestParam X, @ModelAttribute X)
             * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용 
             *
             * @ResponseBody
             * - 메시지 바디 정보 직접 반환(view 조회X)
             * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용 
             */
            @ResponseBody
            @PostMapping("/request-body-string-v4")
            public String requestBodyStringV4(@RequestBody String messageBody) {

                log.info("messageBody={}", messageBody);

                return "ok";

            }

            /**
             * json
             */
            private ObjectMapper objectMapper = new ObjectMapper();

            @PostMapping("/request-body-json-v1")
            public void requestBodyJsonV1(HttpServletRequest request, HttpServletResponse response) throws IOException {

                ServletInputStream inputStream = request.getInputStream();
                String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
                log.info("messageBody={}", messageBody);

                HelloData data = objectMapper.readValue(messageBody, HelloData.class);
                log.info("username={}, age={}", data.getUsername(), data.getAge());

                response.getWriter().write("ok");
            }

            /**
             * @RequestBody
             * HttpMessageConverter 사용 -> StringHttpMessageConverter 적용 *
             * @ResponseBody
             * - 모든 메서드에 @ResponseBody 적용
             * - 메시지 바디 정보 직접 반환(view 조회X)
             * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용 */
            @ResponseBody
            @PostMapping("/request-body-json-v2")
            public String requestBodyJsonV2(@RequestBody String messageBody) throws IOException {

                HelloData data = objectMapper.readValue(messageBody, HelloData.class);
                log.info("username={}, age={}", data.getUsername(), data.getAge());

                return "ok";
            }

            /**
             * @RequestBody 생략 불가능(@ModelAttribute 가 적용되어 버림)
             * HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter (content-type: application/json)
             * requestBodyStringV3와 마찬가지로 HttpEntity로 대체 가능
             */
            @ResponseBody
            @PostMapping("/request-body-json-v3")
            public String requestBodyJsonV3(@RequestBody HelloData data) {
                log.info("username={}, age={}", data.getUsername(), data.getAge());
                return "ok";
            }

            /**
             * @RequestBody 생략 불가능(@ModelAttribute 가 적용되어 버림)
             * HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter (content-type: application/json)
             *
             * @ResponseBody 적용
             * - 메시지 바디 정보 직접 반환(view 조회X)
             * - HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter 적용
            (Accept: application/json)
             */
            @ResponseBody
            @PostMapping("/request-body-json-v5")
            public HelloData requestBodyJsonV5(@RequestBody HelloData data) {
                log.info("username={}, age={}", data.getUsername(), data.getAge());
                return data;
            }
            
        }

```

    - 헤더

```java

        @RestController 
        public class MappingController {
    
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
             * Content-Type 헤더 기반 추가 매핑 Media Type 
             * consumes="application/json"
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

                return "ok";

            }   
        }

```

## 다양한 매핑 응답 방법

    - 정적 리소스

        /static, /public, /resources, /META-INF/resources
            -> 스프링부트가 클래스패스 다음의 위에 해당하는 경로 하위에 존재하는 정적 리소스를 제공함

        src/main/resources/static/basic/hello-form.html 의 파일이 존재하면
        http://localhost:8080/basic/hello-form.html의 URL로 파일 호출 가능

    - 뷰 탬플릿

```java

        @Controller
        public class ResponseViewController {
        
            // ModelAndView를 이용한 응답
            @RequestMapping("/response-view-v1")
            public ModelAndView responseViewV1() {
                ModelAndView mav = new ModelAndView("response/hello")
                        .addObject("data", "hello!");
                return mav;
            }
        
            // Model을 이용한 응답
            @RequestMapping("/response-view-v2")
            public String responseViewV2(Model model) {
                model.addAttribute("data", "hello!!");
                return "response/hello";
            }
            
            // 매핑 URL과 뷰 논리이름이 동일한 경우 뷰 논리이름을 return하지 않아도 알아서 반환해줌
            // 명시적이지 않아 권장하지 않음
            @RequestMapping("/response/hello")
            public void responseViewV3(Model model) {
                model.addAttribute("data", "hello!!");
            }
            
        }

```

    - HTTP 바디

```java

        @Controller
        // @RestController -> @Controller, @ResponseBody를 합친 것과 같음
        // 클래스 레벨에 @ResponseBody를 적용하면 하위의 모든 메서드 레벨에 @ResponseBody를 붙인것과 같음
        // 하위 메서드들이 전부 HTTP 바디로 응답이 나가는 경우 사용
        public class ResponseBodyController {
    
            @GetMapping("/response-body-string-v1")
            public void responseBodyV1(HttpServletResponse response) throws IOException {
                response.getWriter().write("ok");
            }
            
            @GetMapping("/response-body-string-v2")
            public ResponseEntity<String> responseBodyV2() {
                return new ResponseEntity<>("ok", HttpStatus.OK);
            }
            
            @ResponseBody
            @GetMapping("/response-body-string-v3")
            public String responseBodyV3() {
                return "ok";
            }

            @GetMapping("/response-body-json-v1")
            public ResponseEntity<HelloData> responseBodyJsonV1() {
                HelloData helloData = new HelloData();
                helloData.setUsername("userA");
                helloData.setAge(20);
                
                return new ResponseEntity<>(helloData, HttpStatus.OK);
            }
            
            @ResponseStatus(HttpStatus.OK)
            @ResponseBody
            @GetMapping("/response-body-json-v2")
            public HelloData responseBodyJsonV2() {
                HelloData helloData = new HelloData();
                helloData.setUsername("userA");
                helloData.setAge(20);
                
                return helloData;
            }
        }
```

## PRG 패턴

    - POST -> Redirect -> GET 패턴

    - 브라우저 새로고침 시 이전의 POST 요청이 재요청되는 문제점을 방지하기 위함

        단순 조회가 아닌 등록 기능 같이 여러번 작업되면 문제가 발생하기 때문에 방지해야 함

    - POST 요청으로 인한 작업 수행 후 브라우저를 Redirect 시켜 새로고침 하여도 GET방식으로 요청되도록 함

        상품 등록을 위해 POST 요청 -> 상품 목록 화면으로 Redirect -> 새로고침 시 상품 목록 조회를 위한 GET 요청  

### RedirectAttributes

    - Redirect 시 URL 인코딩, PathVariable 바인딩, 쿼리 파라미터 처리를 도와주는 역할

```java

        @PostMapping("/add")
        public String addItemV6(Item item, RedirectAttributes redirectAttributes) {
            Item savedItem = itemRepository.save(item);
            
            redirectAttributes.addAttribute("itemId", savedItem.getId());
            redirectAttributes.addAttribute("status", true);
                
            return "redirect:/basic/items/{itemId}";
        }

```

        return 시에 PathVariable 바인딩 처리없이 문자 연결자(+)를 통해 return 하면 URL 인코딩 처리가 되지 않아 위험함

        addAttribute를 통해 추가된 속성 중 PathVariable 바인딩에 이용되지 않는 속성은 쿼리 파라미터로 처리됨

## RequestMappingHandlerMapping

    - @RequestMapping을 지원하는 핸들러 매핑

    - 스프링 빈 중에서 @RequestMapping 또는 @Controller가 클래스 레벨에 작성되어 있을 경우 핸들러가 취급할 수 있는 매핑 정보로 인식 함 

## RequestMappingHandlerAdapter

    - @RequestMapping을 지원하는 핸들러 어뎁터

    - RequestMappingHandlerAdapter가 실제 핸들러(Controller)를 호출함

        @RequestMapping이 붙은 메서드들을 호출하는 역할을 함
        이 때, 메서드들의 매개변수에 대한 처리는 HandlerMethodArgumentResolver가 수행함

            @RequestParam, @ModelAttribute, @RequestBody, HttpEntity, ...
            수많은 타입의 매개변수들이 존재하고 이에 대한 처리를 위해 
            HandlerMethodArgumentResolver 인터페이스를 구현한 수많은 구현체들이 개입함

## HandlerMethodArgumentResolver

    - 매개변수에 대한 처리를 담당하는 역할을 하는 인터페이스

    - 매개변수 타입에 따라 여러 구현체들이 스프링에 구현되어 있음

    - HandlerMethodArgumentResolver를 줄여서 ArgumentResolver로 불리고 있음

    - 핸들러 어댑터가 핸들러를 호출 할 때 매개변수에 대한 처리 필요 시 HandlerMethodArgumentResolver를 이용

    - supportsParameter() -> 호출된 핸들러의 매개변수에 대한 처리 지원 여부를 판단

    - ResolveArgument() -> 필요에 따라 매개변수 타입의 객체 생성 

## HandlerMethodReturnValueHandler

    - HTTP 요청에 따라 호출된 핸들러의 처리 이후 응답 값의 변환을 담당하는 역할을 하는 인터페이스

    - 응답 타입에 따라 여러 구현체들이 스프링에 구현되어 있음

    - HandlerMethodReturnValueHandler를 줄여서 ReturnValueHandler로 불리고 있음

    - 핸들러 어뎁터가 호출한 핸들러의 응답 값에 대한 처리 필요 시 HandlerMethodReturnValueHandler를 이용

    - supportsReturnType() -> 핸들러의 응답 타입에 대한 처리 지원 여부를 판단

    - handleReturnValue() -> 응답 값의 변환 작업

## HttpMessageConverter

    - 요청과 응답 시에 HTTP 바디에 메시지를 바로 입력하는 경우에 사용됨

        뷰 탬플릿으로 렌더링되어 반환되는 경우에는 ViewResolver가 이용되나 
        HTTP 바디에 메시지로 반환되는 경우에는 ViewResolver 대신 HttpMessageConverter가 이용됨

        HTTP 요청 -> 메서드 레벨에 @RequestBody 또는 매개변수로 HttpEntity(RequestEntity) 이용 시 컨버터 사용
        HTTP 응답 -> 메서드 레벨에 @ResponseBody 또는 매개변수로 HttpEntity(ResponseEntity) 이용 시 컨버터 사용

    - org.springframework.http.converter.HttpMessageConverter 인터페이스

        HTTP 요청을 수용할 수 있는지 판단하는 canRead()와 HTTP 응답이 가능한지 판단하는 canWrite() 기능
        canRead() 후 문제 없을 시 read() 호출, canWrite() 후 문제 없을 시 write() 호출

        스프링에서는 다양한 메시지 컨버터를 제공
    
        대상 클래스 타입과 미디어 타입을 체크하여 사용 가능한 컨버터 결정

    - HttpMessageConverter 우선순위

        ByteArrayHttpMessageConverter > StringHttpMessageConverter > MappingJackson2HttpMessageConverter

        - ByteArrayHttpMessageConverter

            byte[] 데이터 처리
    
            클래스 타입: byte[], 미디어타입: */*(모든 타입 처리 가능)
            요청 예) @RequestBody byte[] data
            응답 예) @ResponseBody return byte[]
                응답 시 Content-Type에 application/octet-stream로 반환함

        - StringHttpMesssageConverter

            String 데이터 처리

            클래스 타입: String, 미디어타입: */*(모든 타입 처리 가능)
            요청 예) @RequestBody String data
            응답 예) @ResponseBody return "..."
                응답 시 Content-Type에 text/plain로 반환함

        - MappingJackson2HttpMessageConverter

            Json 데이터 처리

            클래스 타입: 객체 또는 HashMap , 미디어타입: application/json 관련 타입만 처리 가능
            요청 예) @RequestBody Data data
            응답 예) @ResponseBody return data
                응답 시 Content-Type에 application/json 관련 타입으로 반환함

    - 호출 시점

        HandlerMethodArgumentResolver가 매개변수 타입에 따라 처리를 진행할 때
        매개변수의 타입이 @RequestBody, HttpEntity인 경우 HttpMessageConverter를 호출하여 처리를 위임함

        HandlerMethodReturnValueHandler가 핸들러의 응답 타입에 따라 처리를 진행할 때
        응답 타입이 @ResponseBody, HttpEntity인 경우 HttpMessageConverter를 호출하여 처리를 위임함

        RequestResponseBodyMethodProcessor 
            -> @RequestBody, @ResponseBody에 이용되는 HandlerMethodArgumentResolver

        HttpEntityMethodProcessor
            -> HttpEntity에 이용되는 HandlerMethodArgumentResolver

## WebMvcConfigurer

    - spring mvc에 대한 기능 확장 필요 시 WebMvcConfigurer를 이용하면 됨

매핑 관련 공식 메뉴얼  
[요청 파라미터 목록 공식 메뉴얼](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-arguments)  
[응답 목록 공식 메뉴얼](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-ann-return-types)

