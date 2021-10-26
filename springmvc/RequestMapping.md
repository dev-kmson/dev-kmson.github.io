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


