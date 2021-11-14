---
sort: 5
---

# Bean Validation

---

    - Bean Validation 2.0(JSR-380) 기술 표준

        jakarta.validation-api : Bean Validation 인터페이스
    
    - Bean Validation 기술 표준을 구현한 하이버네이트 validator 구현체가 주로 사용됨

        hibernate-validator : 구현체

    - 검증 어노테이션

        @NotBlank : 빈값, 공백을 허용하지 않음
        @NotNull : null 을 허용하지 않음
        @Range(min = 1000, max = 1000000) : min 미만, max 초과 값을 허용하지 않음 
        @Max(9999) : max의 초과 값을 허용하지 않음

            javax.validation.constraints.NotNull
            org.hibernate.validator.constraints.Range

            javax로 시작하는 것은 특정 구현에 관계없이 제공되는 표준 인터페이스이고,
            org.hibernate.validator로 시작하는 것은 하이버네이트 validator 구현체임
            구현체로 대부분 하이버네이트 validator를 사용함

    - 빈 검증기 직접 사용

```java

    ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
    Validator validator = factory.getValidator();

    Set<ConstraintViolation<Item>> violations = validator.validate(item);

```

        item 객체의 인스턴스 변수에 검증 어노테이션 사용 시
        빈 검증기를 직접 사용하여 검증 어노테이션에 충족하지 못한 경우
        ConstraintViolation을 통하여 검증 오류를 확인할 수 있음

```text

violation={interpolatedMessage='공백일 수 없습니다', propertyPath=itemName, rootBeanClass=class hello.itemservice.domain.item.Item, 
        messageTemplate='{javax.validation.constraints.NotBlank.message}'} 
violation.message=공백일 수 없습니다

violation={interpolatedMessage='9999 이하여야 합니다', propertyPath=quantity, rootBeanClass=class hello.itemservice.domain.item.Item, 
        messageTemplate='{javax.validation.constraints.Max.message}'} 
violation.message=9999 이하여야 합니다

violation={interpolatedMessage='1000에서 1000000 사이여야 합니다', propertyPath=price, rootBeanClass=class hello.itemservice.domain.item.Item, 
        messageTemplate='{org.hibernate.validator.constraints.Range.message}'} 
violation.message=1000에서 1000000 사이여야 합니다

```

하이버네이트 Validator 관련 링크
[공식 사이트](http://hibernate.org/validator/)
[공식 메뉴얼](https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/)
[검증 어노테이션 모음](https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/#validator-defineconstraints-spec)

## Spring - BeanValidator

    - Spring과 BeanValidator의 통합

```java

    @Data
    public class Item {
    
        private Long id;
    
        @NotBlank
        private String itemName;
    
        @NotNull
        @Range(min = 1000, max = 1000000)
        private Integer price;
    
        @NotNull
        @Range(min = 1, max = 9999)
        private Integer quantity;
    
        public Item() {
        }
    
        public Item(String itemName, Integer price, Integer quantity) {
            this.itemName = itemName;
            this.price = price;
            this.quantity = quantity;
        }
    }

```

```java
    
    @PostMapping("/add")
    public String addItem(@Validated @ModelAttribute Item item, BindingResult
            bindingResult, RedirectAttributes redirectAttributes) {
    
            ...
    }

```

        spring boot는 spring-boot-starter-validation 라이브러리 존재 시 Bean Validator를 인지하여 
        LocalValidatorFactoryBean을 global Validator로 등록하고 검증 어노테이션을 인식하여 검증을 수행함

            검증 처리기를 따로 등록하지 않아도 global Validator가 등록되어 있기 때문에
            @Validated 또는 @Valid 적용 시 처리 가능

                @Validated : 스프링 전용 검증 어노테이션
                @Valid : 자바 표준 검증 어노테이션

        즉, 매핑되는 메서드에 @Validated가 있으면 검증 오류를 처리하기 위해
        매핑되는 메서드의 모델 객체를 처리할 수 있는 검증 처리기를 탐색하게 되고
        global Validator로 등록된 LocalValidatorFactoryBean가 검증 처리기로써
        역할을 하여 검증 오류 발생 시 내부적으로 FieldError, ObjectError을 생성하여
        BindResult에 담아줌

    - 검증 순서

        1. 모델 객체의 각각 필드에 타입 변환 시도
            
            타입 변환 실패 시 스프링이 typeMismatch 에러코드로 FieldError을 추가

        2. 타입 변환을 성공한 필드에 대해서만 Bean Validation을 적용
                
            LocalValidatorFactoryBean이 FieldError, ObjectError 생성 및 BindResult에 추가

## errorCode

    - MessageCodesResolver에 의한 메시지 키 자동 생성

        BindingResult의 rejectValue(), reject()에 의하여 MessageCodesResolver가 호출되어
        메시지 키를 자동 생성한 것과 같이 Bean Validation 또한 rejectValue(), reject()를 통해
        검증 어노테이션마다 MessageCodesResolver를 호출하여 메시지 키를 자동으로 생성함

```text

        codes [NotBlank.item.itemName,NotBlank.itemName,NotBlank.java.lang.String,NotBlank]
        codes [Range.item.price,Range.price,Range.java.lang.Integer,Range]

```

        메시지 키가 자동으로 생성되었으므로 errors.properties에 메시지 키에 대응되는 값을 작성하여
        개발자가 작성한 오류 메시지를 전달할 수 있음

```properties

        # errors.properties Bean Validation 추가 
        
        NotBlank.item.itemName=상품 이름을 입력하세요
        
        NotBlank={0} 공백X
        Range={0}, {2} ~ {1} 허용
        Max={0}, 최대 {1}

```

        Bean Validation의 메시지 탐색 순서

            MessageCodesResolver에 의하여 생성된 키의 순서대로 messageSource 확인
            검증 어노테이션에 설정된 message 옵션의 값 확인
            라이브러리에서 제공하는 기본 값 사용
        
## ObjectError 처리

    - @ScriptAssert를 통한 ObjectError 처리

        검증 어노테이션은 필드 단위로 작성되므로 ObjectError의 처리 방법이 따로 필요함
        
```java

        @Data
        @ScriptAssert(lang = "javascript", script = "_this.price * _this.quantity >= 10000", 
                message = "총합이 10000원 이상이어야 합니다")
        public class Item {
            //...
        }

```

```text

        codes [ScriptAssert.item,ScriptAssert]

```

        @ScriptAssert 작성 시 ObjectError 처리 및 메시지 키 자동 생성 등 여러 기능을 제공해주나
        실무에서는 검증 기능이 해당 객체의 범위를 벗어나는 등 제약이 많고 복잡함

        따라서, ObjectError의 경우 @ScriptAssert를 억지로 사용하기 보다는 기존 자바 코드 작성을
        통해 해결하는 것이 권장됨

## groups

    - 같은 모델 객체의 등록 시 요구사항과 수정 시 요구사항이 다른 경우 각각 그룹으로 나누어 적용

        그룹을 적용하기 위해 각각 등록용 인터페이스와 수정용 인터페이스를 생성

```java

        public interface SaveCheck {}
        
        public interface UpdateCheck {}

```

        검증 어노테이션에 어느 그룹에 적용될 어노테이션인지를 명시함

```java

        @Data
        public class Item {
            @NotNull(groups = UpdateCheck.class) 
            private Long id;
            
            @NotBlank(groups = {SaveCheck.class, UpdateCheck.class})
            private String itemName;
            
            @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
            @Range(min = 1000, max = 1000000, groups = {SaveCheck.class, UpdateCheck.class})
            private Integer price;
            
            @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
            @Max(value = 9999, groups = SaveCheck.class)  
            private Integer quantity;
            
            public Item() {}
            
            public Item(String itemName, Integer price, Integer quantity) {
                this.itemName = itemName;
                this.price = price;
                this.quantity = quantity;
            } 
        }

```

        @Validated 옵션을 통하여 그룹을 선택할 수 있음

            @Valid에는 그룹을 선택할 수 있는 옵션이 존재하지 않음
            groups 기능을 이용하려면 @Validated를 사용하여야 함

```java

        @PostMapping("/add")
        public String addItemV2(@Validated(SaveCheck.class) @ModelAttribute Item item, 
                BindingResult bindingResult, RedirectAttributes redirectAttributes) {
            //...
        }

```

        groups 기능을 이용하여 특정 기능에 따라 검증 적용을 다르게 할 수 있지만
        인터페이스 생성과 그룹 지정 등 복잡도 증가

        실무에서는 등록용 폼 객체와 수정용 폼 객체를 분리해서 사용하기 때문에
        실제로 groups 기능은 잘 사용되지 않음

## form 객체 분리

    - 동일한 모델 객체가 아닌 폼에 따른 객체를 분리

        등록과 수정 폼에 따라 필요로 하는 데이터가 서로 다르기 때문에 
        동일한 모델 객체를 이용하지 않고 서로 다른 객체로 관리하는 것이 좋음
    
            Item 모델 객체를 ItemSaveForm, ItemUpdateForm 으로 분리
            
                일반적으로 ...From, ...Request, ...Dto 등의 네이밍 사용

        기존 Item 모델 객체에 존재하던 검증 어노테이션을 등록 폼과 수정 폼의 요구 사항에 맞게
        구분하여 검증 어노테이션을 적용함 

```java

        @PostMapping("/add")
        public String addItem(@Validated @ModelAttribute ItemSaveForm form,
            BindingResult bindingResult, RedirectAttributes redirectAttributes){
    
                //...

                Item item = new Item();
                item.setItemName(form.getItemName());
                item.setPrice(form.getPrice());
                item.setQuantity(form.getQuantity());
                
                //...
        }

```

        모델 객체로 기능에 맞는 폼 객체를 이용
        
            모델 객체로 별도의 폼 객체를 두었기 때문에 도메인 객체인 Item 객체를 
            생성 및 변환해주는 과정이 추가됨

## RequestBody

    - @RequestBody에 Bean Validation 적용

        @ModelAttribute와 마찬가지로 @RequestBody 또한 @Valid, @validated를 적용할 수 있음
        단, @ModelAttribute와는 다르게 타입 변환 실패 시 컨트롤러 자체가 호출되지 못하는 현상을
        방지 할 수 없음

            @ModelAttribute는 필드 단위로 정교하게 바인딩이 적용되어 특정 필드 바인딩이 실패하더라도
            나머지 필드는 정상적으로 바인딩되나 @RequestBody에 이용되는 HttpMessageConverter는
            JSON 데이터를 객체로 변경하지 못하면 이후 단계가 진행되지 못하고 예외가 발생함