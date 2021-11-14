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
        


