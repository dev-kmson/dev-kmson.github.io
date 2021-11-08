---
sort: 4
---

# Validation

---

## BindingResult

    - 스프링이 제공하는 검증 오류 처리를 위한 보관 객체 

        상황에 따른 필드 별 오류 문구뿐만 아니라 잘못된 타입 요청에 대한 처리 등
        Validation을 유용하게 처리할 수 있도록 도와줌

        BindingResult는 인터페이스로써, Error 인터페이스를 상속받고 있음
        구현체는 BeanPropertyBindingResult로써 BindingResult, Error 인터페이스를
        모두 구현하고 있음

        BindingResult 대신 Error를 사용해도 되나 Errors는 단순한 오류 저장과 
        조회 기능만을 제공하므로 보통 BindingResult를 사용함

        BindResult없이 @ModelAttribute만 사용 시 타입 오류로 인해 바인딩을 실패하면
        해당 컨트롤러가 호출되지 않고 오류페이지로 이동함
        반면, BindResult가 있으면 어떤 필드에서 어떤 타입 오류로 인하여 바인딩이 실패하였는지
        FieldError에 스프링이 오류 내용을 자동적으로 넣어줌
        따라서, 사용자가 오류페이지가 아닌 실패 내용을 확인할 수 있음
        

```java

        @PostMapping("/add")
        public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, 
                RedirectAttributes redirectAttributes) {
        
            if (!StringUtils.hasText(item.getItemName())) { 
                bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수입니다.")); 
            }
        
            if (item.getPrice() != null && item.getQuantity() != null) {
                int resultPrice = item.getPrice() * item.getQuantity();
                
                if (resultPrice < 10000) {
                    bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
                } 
            }
            
        }

```

        BindingResult는 반드시 @ModelAttribute 다음 순서에 위치하여야 함
        
        BindingResult는 Model에 자동으로 포함되기 때문에 따로 Model에 담을 필요가 없음

        특정 필드에 대한 오류는 FieldError를 이용하고 특정 필드의 오류가 아닌 경우에는 ObjectError를 이용

## ThymeLeaf #fields

    - 타임리프는 스프링의 BindingResult를 활용해서 편리하게 검증 오류를 표현하는 기능을 제공함

```html

        <div th:if="${#fields.hasGlobalErrors()}">
            <p class="field-error" th:each="err : ${#fields.globalErrors()}" th:text="${err}">글로벌 오류 메시지</p> 
        </div>

```

        ${#fields.hasGlobalErrors()}를 통하여 BindingResult의 ObjectError 목록을 얻어올 수 있음

```html

<input type="text" id="itemName" th:field="*{itemName}" 
       th:errorclass="field-error" class="form-control" placeholder="이름을입력하세요">
<div class="field-error" th:errors="*{itemName}">
    상품명 오류
</div>

```

        해당 field와 같은 이름을 가진 FieldError가 있으면 오류 내용을 표현
        
            th:errorclass는 설정한 클래스를 classappend 형식으로 렌더링
            th:errors는 FieldError에 보관된 defaultMessage를 출력
            th:field는 FieldError에 보관된 rejectedValue를 출력

                th:field는 정상적인 상황이면 모델 객체의 값을 사용하지만, 오류가 발생하면 FieldError에 보관된 값을 사용함
                따라서, FieldError에 rejectedValue 파라미터를 작성하지 않으면 빈값이 출력되어 사용자가 입력한 값이
                보관되지 않음

## FieldError

    - 파라미터 목록

        objectName : 오류가 발생한 객체 이름
        field : 오류 필드
        rejectedValue : 사용자가 입력한 값(거절된 값)
        bindingFailure : 타입 오류 같은 바인딩 실패인지, 검증 실패인지에 대한 구분 값
        codes : 메시지 코드
        argument : 메시지에서 사용되는 인자
        defaultMessage : 기본 오류 메시지

## 오류 메시지 처리

### version1

    - errors.properties를 통한 오류 메시지 별로 관리

        application.properties에 errors 추가

        messages와 마찬가지로 국제화 처리 가능

```properties

        required.item.itemName=상품 이름은 필수입니다.
        range.item.price=가격은 {0} ~ {1} 까지 허용합니다.
        max.item.quantity=수량은 최대 {0} 까지 허용합니다.
        totalPriceMin=가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}

```        

```java
        @PostMapping("/add")
        public String addItemV3(@ModelAttribute Item item, BindingResult bindingResult, 
                RedirectAttributes redirectAttributes, Model model) {
    
            if(!StringUtils.hasText(item.getItemName())) {
                bindingResult.addError(
                        new FieldError("item", "itemName", item.getItemName(), false, 
                                new String[]{"required.item.itemName"}, null, null));
            }
            if(item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
                bindingResult.addError(
                        new FieldError("item", "price", item.getPrice(), false, 
                                new String[]{"range.item.price"}, new Object[]{1000, 1000000}, null));
            }
            if (item.getQuantity() == null || item.getQuantity() >= 9999) {
                bindingResult.addError(
                        new FieldError("item", "quantity", item.getQuantity(), false, 
                                new String[]{"max.item.quantity"}, new Object[]{9999}, null));
            }
    
            if (item.getPrice() != null && item.getQuantity() != null) {
                int resultPrice = item.getPrice() * item.getQuantity();
                if (resultPrice < 10000) {
                    bindingResult.addError(
                        new ObjectError("item", 
                                new String[]{"totalPriceMin"}, new Object[]{10000, resultPrice}, null));
                }
            }
            
            ...
        }

```

        FieldError, ObjectError를 생성하기 위해 타겟 객체를 일일이 지정,
        오류 내용을 표현하기 위해 FieldError 생성, ObjectError 생성, 
        errors.properties의 오류 내용을 가져오기 위해 String, Object 배열을 생성
        위와 같이 작성하여야할 것이 많아지는 불편함 존재

### version2

    - BindingResult를 이용할 때 target 객체를 미리 지정하는 것이 번거로움

        어차피 BindingResult는 @ModelAttribute 다음 순서에 위치하므로 target을 유추할 수 있음

            bindingResult.getObjectName() -> 모델에 담긴 객체의 변수명
            bindingResult.getTarget() -> 모델에 담긴 실제 객체
        
        FieldError과 ObjectError를 직접 생성하지 않고 검증 오률를 다룰 수 있음
        
            bindingResult.rejectValue() -> FieldError 대체
            bindingResult.reject() -> ObjectError 대체

                rejectValue(), reject()의 내부에서 직접 FieldError, ObjectError을 생성하고 있음을 확인할 수 있음

```java

        @PostMapping("/add")
        public String addItemV4(@ModelAttribute Item item, BindingResult bindingResult, 
                RedirectAttributes redirectAttributes, Model model) {
    
            if(!StringUtils.hasText(item.getItemName())) {
                bindingResult.rejectValue("itemName", "required");
            }
            if(item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
                bindingResult.rejectValue("price", "range", new Object[]{1000, 1000000}, null);
            }
            if (item.getQuantity() == null || item.getQuantity() >= 9999) {
                bindingResult.rejectValue("quantity", "max", new Object[]{9999}, null);
            }
    
            if (item.getPrice() != null && item.getQuantity() != null) {
                int resultPrice = item.getPrice() * item.getQuantity();
                if (resultPrice < 10000) {
                    bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
                }
            }
            
            ...
        }

```        

        위의 errors.properties를 통한 메시지 처리 방법은 메시지가 너무 구체적이여서
        사용할 수 있는 범위가 한정적이라는 단점이 존재함
        따라서, 메시지를 범용적으로 사용할 수 있게 추상화 시키고 특정한 경우 세밀하게
        내용이 작성되어야 하는 경우에는 구체적 메시지가 적용될 수 있도록 하는 것이 가장 좋은 방법임

        MessageCodesResolver가 타겟 정보를 이용하여 메시지 키에 대한 우선순위를 제공함

```java
    
        MessageCodesResolver codesResolver = new DefaultMessageCodesResolver();

        @Test
        void messageCodesResolverObject() {
            String[] messageCodes = codesResolver.resolveMessageCodes("required", "item", "itemName", String.class);
            System.out.println("messageCodes = " + Arrays.toString(messageCodes));
        }

```

```text

        messageCodes = [required.item.itemName, required.itemName, required.java.lang.String, required]

```

        MessageCodesResolver 테스트 결과 resolveMessageCodes()가 타겟 정보를 이용해
        메시지 키의 우선순위를 배열로 반환했음을 확인할 수 있음

        rejectValue(), reject()가 FiledError와 ObjectError을 생성하면서 resolveMessageCodes()의
        반환 정보를 매개변수로 이용함
        
        타임리프의 th:errors가 렌더링되면서 MessageSource를 통해 properties 파일을 읽어
        실제 저장된 오류 내용을 표출함

## ValidationUtils

```java

if (!StringUtils.hasText(item.getItemName())) { 
    bindingResult.rejectValue("itemName", "required", "기본: 상품 이름은 필수입니다."); 
}

// ValidationUtils를 이용하여 간단히 표현할 수 있음

ValidationUtils.rejectIfEmptyOrWhitespace(bindingResult, "itemName", "required");

```

## typeMismatch

    - 스프링이 처리해주는 오류 메시지

        타입이 불일치할 경우 스프링이 typeMismatch라는 에러코드를 가진 FieldError를 생성함

```text

        codes[typeMismatch.item.price,typeMismatch.price,typeMismatch.java.lang.Integer,typeMismatch]
    
```

        스프링은 타입 오류가 발생하면 typeMismatch 라는 오류 코드를 사용 
        이 오류 코드가 MessageCodesResolver를 통하여 메시지 키를 생성한 것

        개발자가 직접 errors.properties에 메시지를 정의하여 오류 메시지를 처리하기도 하지만
        타입 불일치와 같이 스프링이 직접 처리하는 오류 메시지의 경우 디폴트로 작성되어 있는 메시지는
        일반 사용자에게 보여주기에는 무리가 있는 메시지로 작성되어 있음

```text

        Failed to convert property value of type java.lang.String to required type
        java.lang.Integer for property price; nested exception is
        java.lang.NumberFormatException: For input string: "A"

```

        이러한 불친절한 메시지를 사용자에게 노출시키지 않기 위해서 개발자가 메시지를 정의해 주어야 함

```properties

        typeMismatch.java.lang.Integer=숫자를 입력해주세요.
        typeMismatch=타입 오류입니다.

```

## Validator

### version1

    - 검증 로직으로 인해 컨트롤러의 핵심 로직이 명확히 보이지 않아 검증 로직을 별도 역할로 분리


```java

    public class ItemValidator {

        public void validate(Object target, Errors errors) {
            ...
        }
        
    }

```

        validate() 안에 검증 로직을 구현

            validate()는 Object target, Errors errors 매개변수를 가지는데
            target에는 검증할 모델 객체를 넘기고 errors에는 BindingResult를 넘김

        직접 Validator 클래스를 작성하고 검증 로직을 validate() 안에 
        구현 후 클래스를 인스턴스화하여 검증 기능을 수행할 수도 있지만
        스프링에서 제공하는 Validator 인터페이스를 이용하면
        컨트롤러에서 검증기를 자동으로 적용할 수 있음

### version2

    - 스프링에서 제공하는 Validator 이용

```java
    
    // Validator

    @Component
    public class ItemValidator implements Validator {
    
        @Override
        public boolean supports(Class<?> clazz) {
            return Item.class.isAssignableFrom(clazz);
        }
    
        @Override
        public void validate(Object target, Errors errors) {
            ...
        }
    }

```

        스프링이 제공하는 Validator 인터페이스를 구현

        supports : 해당 검증기를 지원하는 여부 확인
        validate : 검증 로직 구현

```java

    // Controller
    
    private final ItemValidator itemValidator;
    
    @InitBinder
    public void init(WebDataBinder dataBinder) {
        dataBinder.addValidators(itemValidator);
    }

    @PostMapping("/add")
    public String addItemV6(@Validated @ModelAttribute Item item, 
            BindingResult bindingResult, RedirectAttributes redirectAttributes){
        
        ...
    }
```

        @InitBinder와 WebDataBinder를 이용하여 itemValidator에 대한 검증기를 자동으로 적용 
        따라서, 어느 매핑 메서드에서든 검증기를 실행할 수 있음        

        매핑 메서드에 @Validated는 검증기를 실행하라는 의미
        WebDataBinder에 등록한 검증기를 찾아서 실행함
        이 때, 검증기는 여러개가 등록될 수 있으므로 어떤 검증기를 실행한 것인지에 대한 구분이 필요함
        이 구분은 Validator 인터페이스의 supports가 기능을 수행하며
        @ModelAttribute의 객체 타입을 검증기로 지원하는지 체크 후 지원하지 않는 검증기이면
        등록되어 있는 다른 검증기에서 다시 supports를 수행하여 지원하는 검증기를 탐색함
        
    - global 설정

        모든 컨트롤러에서 Validator가 적용될 수 있도록 다음과 같이 설정할 수 있음

```java

    @SpringBootApplication
    public class ItemServiceApplication implements WebMvcConfigurer {
        
        public static void main(String[] args) {
            SpringApplication.run(ItemServiceApplication.class, args);
        }
        
        @Override
        public Validator getValidator() {
            return new ItemValidator();
        }
    }

```

