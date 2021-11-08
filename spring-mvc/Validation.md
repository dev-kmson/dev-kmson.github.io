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