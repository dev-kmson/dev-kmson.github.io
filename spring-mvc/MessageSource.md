---
sort: 3
---

# MessageSource

---

    - 메시지
        
        스프링에서는 메시지 관리 기능을 위해 MessageSource 인터페이스를 제공함

        스프링 부트는 MessageSource 인터페이스를 구현한 ResourceBundleMessageSource 구현체를
        스프링 빈으로 등록하여 메시지 관리 기능을 이용할 수 있도록 함

        ResourceBundleMessageSource는 기본적으로 basename을 messages로 설정하기 때문에 
        messages.properties 파일을 이용하여 메시지를 관리함

            디폴트로 설정되는 messages 대신 다른 설정 파일을 basename으로 지정하고 싶으면
            application.properties에 spring.messages.basename에 설정 파일명을 기입

```properties

            # default
            spring.messages.basename=messages

```


        어플리케이션 실행 중에 메시지 파일을 변경하여 인식할 수 있도록 하기 위해서는
        ReloadableResourceBundleMessageSource 구현체를 이용


    - 국제화

        설정된 언어에 따라 메시지를 달리 보여주기 위해서 스프링에서 국제화 기능을 제공함
        
        스프링 부트에서 제공하는 ResourceBundleMessageSource를 이용할 때는
        basename에 _를 붙인 후 ko, en 과 같이 언어를 붙인 파일을 생성 및 작성하면 됨

            만일 설정한 언어에 해당하는 국제화 파일을 찾을 수 없으면 단순 basename으로 된
            국제화 파일이 이용 됨

    - 메시지 사용

```java

// properties 파일에 설정된 키 hello에 등록된 값이 출력됨
@Test
void helloMessage() {
    String result = ms.getMessage("hello", null, null); 
    assertThat(result).isEqualTo("안녕");
}

// properties 파일에 등록되지 않은 키 이용 시 NoSuchMessageException 발생
@Test
void notFoundMessageCode() {
    assertThatThrownBy(() -> ms.getMessage("no_code", null, null))
            .isInstanceOf(NoSuchMessageException.class);
}

// properties 파일에 등록되지 않은 키 이용 시 default 메시지를 설정하여 출력할 수 있음
@Test
void notFoundMessageCodeDefaultMessage() {
    String result = ms.getMessage("no_code", null, "기본 메시지", null);
    assertThat(result).isEqualTo("기본 메시지"); 
}

// properties 파일에 {0}, {1} ... 과 같이 매개변수를 받을 수 있도록 설정할 수 있고
// Object 배열을 통하여 매개변수를 전달할 수 있음
@Test
void argumentMessage() {
    String result = ms.getMessage("hello.name", new Object[]{"Spring"}, null); 
    assertThat(result).isEqualTo("안녕 Spring");
}

// Locale을 설정하지 않으면 디폴트 국제화 파일을 이용하고 Locale 설정 시 해당 언어에 맞는 국제화 파일을 이용함
@Test
void enLang() {
    assertThat(ms.getMessage("hello", null, Locale.ENGLISH)).isEqualTo("hello");
}

```

    - ThymeLeaf를 이용한 메시지 관리

        #{...} 문법을 이용하여 국제화 파일에 등록된 키 작성 시 국제화 파일에 등록되어 있는 값을 불러올 수 있음

        매개변수 필요 시 #{...(${...})}과 같이 매개변수 전달 가능

```html

<th th:text="#{label.item.id}">ID</th>
<th th:text="#{label.item.itemName}">상품명</th> 
<th th:text="#{label.item.price}">가격</th>
<th th:text="#{label.item.quantity}">수량</th>

<th th:text="#{hello.name(${item.itemName})}"></th>


```

        스프링은 Locale 정보를 이용하여 어떤 언어를 선택할 것인지 판별함

        Locale 선택 방식을 변경할 수 있도록 LocaleResolver 인터페이스 제공

        스프링 부트는 기본적으로 'Accept-Language' 헤더 정보를 활용하는 
        AcceptHeaderLocaleResolver 구현체를 사용함

            ThymeLeaf 이용 시 웹 브라우저에서 전달한 헤더의 Accept-Language의 
            언어 우선순위 설정 정보에 따라 국제화 파일이 적용되어 값이 표출됨

        


