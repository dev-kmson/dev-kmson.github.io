---
sort: 1
---

# ThymeLeaf

[기본 메뉴얼](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html)  
[스프링 통합 메뉴얼](https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html)

[템플릿 엔진 설정](https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html#the-springstandard-dialect)  
[뷰 리졸버 설정](https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html#views-and-view-resolvers)

---

## text, utext

```html

    <li>th:text = <span th:text="${data}"></span></li>
    <li>th:utext = <span th:utext="${data}"></span></li>

    <li><span th:inline="none">[[...]] = </span>[[${data}]]</li>
    <li><span th:inline="none">[(...)] = </span>[(${data})]</li>

```

    - th:text -> escape 처리, th:utext -> excape 처리 x 

        escape : html 문자로 인식되지 않도록 단순 문자로 처리되도록 함

    - [[${data}]] -> th:text와 동일, [(${data})] -> th:utext와 동일

## SpringEL 표현식

    - Object

```html

    <span th:text="${user.username}"></span></li>
    <span th:text="${user['username']}"></span></li>
    <span th:text="${user.getUsername()}"></span></li>

```

    - List

```html

    <li>${users[0].username} = <span th:text="${users[0].username}"></span></li>
    <li>${users[0]['username']} = <span th:text="${users[0]['username']}"></span></li>
    <li>${users[0].getUsername()} = <span th:text="${users[0].getUsername()}"></span></li>

```

    - Map

```html

    <li>${userMap['userA'].username} =  <span th:text="${userMap['userA'].username}"></span></li>
    <li>${userMap['userA']['username']} = <span th:text="${userMap['userA']['username']}"></span></li>
    <li>${userMap['userA'].getUsername()} = <span th:text="${userMap['userA'].getUsername()}"></span></li>

```

    - 지역 변수 선언

```html

    <div th:with="first=${users[0]}">
        <span th:text="${first.username}"></span>
    </div>

```

        th:with 속성이 들어간 태그 및 하위 영역에서만 사용 가능

## 기본 객체

    - Controller단에서 model로 넘겨주지 않아도 타임리프가 제공하는 객체

        ${#request}
        ${#response}
        ${#session} 
        ${#servletContext} 
        ${#locale}

        ${param.~~~} : 쿼리 파라미터를 담아둔 객체
        ${@helloBean.~~~} : 스프링 빈을 담아둔 객체

## 유틸리티 객체

    - 숫자 서식, 배열, 날짜, 자바8 날짜, 컬렉션 등 유틸리티 관련하여 타임리프가 제공하는 객체

[타임리프 유틸리티 객체](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#expression-utility-objects)  
[유틸리티 객체 예시](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#appendix-b-expression-utility-objects)

## URL 링크

    - @{...} 문법을 통해 링크 부여

```html

    <li><a th:href="@{/hello}">basic url</a></li>
    <li><a th:href="@{/hello(param1=${param1}, param2=${param2})}">hello query param</a></li>
    <li><a th:href="@{/hello/{param1}/{param2}(param1=${param1}, param2=${param2})}">path variable</a></li>
    <li><a th:href="@{/hello/{param1}(param1=${param1}, param2=${param2})}">path variable + query parameter</a></li>

```

        ()안에 내용이 쿼리 파라미터로 만들어짐
        
        PathVariable 링크 부여 시 ()안에 내용이 PathVariable에 바인딩 됨

        PathVariable로 명시되지 않은 내용이 있다면 자동적으로 쿼리 파라미터로 만들어짐 

[절대 경로, 상대 경로, 프로토콜 관련](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#link-urls)

## 리터럴 주의사항

    - 문자는 ''로 감싸야 문자로 인식됨

```html

    <span th:text="'hello'">

```

        예외적으로 공백이 없을 시 문자로 인식

```html

    <span th:text="hello"> 
    <span th:text="hello world!"></span>
```

        "hello"는 가능하나 "hello world!"는 문자로 인식되지 않아 오류 발생

```html

    <span th:text="|hello ${data}|">

```

        || 사이에 작성 시 ''없이 문자로 인식 가능, SpringEL 표현식도 함께 작성 가능 

## 연산

    - 비교 연산자 목록 

        > (gt)
        < (lt)
        >= (ge)
        <= (le)
        ! (not)
        == (eq)
        != (neq, ne)

    - 조건식 사용

```html

    <span th:text="(10 % 2 == 0)?'짝수':'홀수'"></span>

```

    - Elvis 연산자

```html

    <span th:text="${data}?: '데이터가없습니다.'"></span>

```

    SpringEL 표현식으로 가져온 값이 null인 경우 ''안의 내용으로 대체

    - No-Operation

```html

    <span th:text="${data}?: _">데이터가 없습니다.</span>

```

    SpringEL 표현식으로 가져온 값이 null인 경우 타임리프가 실행되지 않는 것 처럼 동작

## 속성 값 설정

    - th:* 를 이용하여 속성 부여
        
        태그 안에 동일한 속성 존재 시 타임리프로 부여한 속성으로 대체됨    

```html

        <input type="text" class="text" th:attrappend="class=' large'" /><br/>
        <input type="text" class="text" th:attrprepend="class='large '" /><br/>
        <input type="text" class="text" th:classappend="large" / ><br/>

```

        th:attrappend : 속성 값의 뒤에 값을 추가 -> class="text large"
        th:attrprepend : 속성 값의 앞에 값을 추가 -> class="large text"
        th:classappend : class 속성에 값 추가 -> class="text large"

    - checked

```html

        <input type="checkbox" name="active" th:checked="false" /><br/>

```

        html에서는 checkbox 타입의 input 태그에 checked 속성이 존재하면
        속성의 값 여부와 상관없이 무조건 체크박스를 체크상태로 둠
        따라서, 체크를 해제하기 위해 상황에 따라 checked 속성을 삭제 혹은 추가하는
        불편한 작업을 해야했음
        
        th:checked로 속성 부여 시 true면 체크된 상태로 
        false면 체크가 되지 않는 상태로 만들어줌

## 반복

    - th:each

```html

    <tr th:each="user, userStat : ${users}">
        <td th:text="${userStat.count}">username</td>
        <td th:text="${user.username}">username</td>
        <td th:text="${user.age}">0</td>
        <td>
            index = <span th:text="${userStat.index}"></span>
            count = <span th:text="${userStat.count}"></span>
            size = <span th:text="${userStat.size}"></span>
            even? = <span th:text="${userStat.even}"></span>
            odd? = <span th:text="${userStat.odd}"></span>
            first? = <span th:text="${userStat.first}"></span>
            last? = <span th:text="${userStat.last}"></span>
            current = <span th:text="${userStat.current}"></span>
        </td> 
    </tr>

```

        th:each="user, userStat : ${users}"

        반복 상태를 제공하는 두번째 파라미터는 생략 시 
        변수명 + 'Stat'으로 기본 제공되므로 디폴트 이름
        그대로 사용 시에는 생략 가능

## 조건부 평가

    - if, unless

```html

    <span th:text="'미성년자'" th:if="${user.age lt 20}"></span> 
    <span th:text="'미성년자'" th:unless="${user.age ge 20}"></span>

```

        unless -> if의 반대

        조건을 만족하지 않을 시 태그 자체를 렌더링 하지 않음
        즉, span태그 자체가 사라짐

    - switch

```html

    <td th:switch="${user.age}">
        <span th:case="10">10살</span>
        <span th:case="20">20살</span> 
        <span th:case="*">기타</span>
    </td>

```

## 주석

    - html 주석

```html

    <!--
    <span th:text="${data}">html data</span> 
    -->

```

        타임리프가 html 주석은 처리하지 않고 그대로 두기 때문에
        렌더링 후 html 소스 확인 시 주석이 그대로 남아있음

    - 타임리프 파서 주석

```html

    <!--/*-->
    <span th:text="${data}">html data</span>
    <!--*/-->

```

        타임리프용 주석으로써 타임리프 렌더링 후에는 해당 주석이 사라짐
        렌더링 후 html 소스 확인 시 주석이 사라져있음

    - 타임리프 프로토타입 주석

```html
    
    <!--/*/
    <span th:text="${data}">html data</span> 
    /*/-->

```

        앞과 뒤가 html 주석과 일치하기 때문에 html 파일 자체를
        열어볼 때는 주석으로써 보여지지만 타임리프 렌더링 후에는
        주석처리 되지않은 정상적인 태그로 표출됨

## 블록

    - 블록

```html

    <th:block th:each="user : ${users}">
        <div>
            사용자 이름1 <span th:text="${user.username}"></span>
            사용자 나이1 <span th:text="${user.age}"></span> </div>
        <div>
            요약 <span th:text="${user.username} + ' / ' + ${user.age}"></span>
    
        </div>
    </th:block>

```

        th:each로 반복 시 태그 내에서 반복 기능이 수행되기 때문에
        여러 태그를 반복하고 싶을 때 용이하지 않음

        타임리프에서 th:block를 통하여 여러 태그를 반복할 수 있도록 제공함

## 자바스크립트 인라인

    - script 태그 안의 자바스크립트 소스를 편리하게 작성할 수 있도록 함

        텍스트 렌더링 :

            var username = [[${user.username}]];
            인라인 사용 전 var username = userA; 
            인라인 사용 후 var username = "userA";

            문자 리터럴로 감싸주지 않아도 타임리프가 렌더링 시 문자로 처리함

        자바스크립트 내추럴 템플릿

            var username2 = /*[[${user.username}]]*/ "test username"; 
            인라인 사용 전 var username2 = /*userA*/ "test username"; 
            인라인 사용 후 var username2 = "userA";

            html뿐만 아니라 자바스크립트도 내추럴 템플릿처럼 작용할 수 있게 함

        JSON

            var user = [[${user}]];
            인라인 사용 전 var user = BasicController.User(username=userA, age=10); 
            인라인 사용 후 var user = {"username":"userA","age":10};

            렌더링 시 JSON 형식으로 바꾸어 값을 할당함 

    - 자바스크립트 반복

```html

    <script th:inline="javascript">
        [# th:each="user, stat : ${users}"]
        var user[[${stat.count}]] = [[${user}]];
        [/]
    </script>

```

        인라인 적용 시 스크립트 태그 안에서 타임리프 반복 기능을 제공함

## 템플릿 레이아웃

    - 화면 레이아웃을 공통적으로 가져가고 특정한 부분들을 주입 받아 화면을 구성할 수 있음

```html

    <!DOCTYPE html>
    <html th:fragment="layout (title, content)" xmlns:th="http://www.thymeleaf.org">
    <head>
        <title th:replace="${title}">레이아웃 타이틀</title> </head>
    <body>
    <h1>레이아웃 H1</h1>
    <div th:replace="${content}"> 
        <p>레이아웃 컨텐츠</p>
    </div>
    <footer> 
        레이아웃 푸터
    </footer>
    </body>
    </html>
    
```

```html
    
    <!DOCTYPE html>
      <html th:replace="~{template/layoutExtend/layoutFile :: layout(~{::title}, ~{::section})}" xmlns:th="http://www.thymeleaf.org">
      <head>
    <title>메인 페이지 타이틀</title> </head>
      <body>
      <section>
        <p>메인 페이지 컨텐츠</p>
        <div>메인 페이지 포함 내용</div> 
      </section>
      </body>
      </html>

```

        th:fragment -> th:replace, th:insert로 대체 또는 삽입되어질 대상, 메서드처럼 인자를 받을 수 있음

        th:replace -> fragment 지정 또는 fragment의 인자로 대상 태그를 대체함 
        th:insert ->  fragment 지정 또는 fragment의 인자로 대상 태그의 하위에 삽입함

            fragment 지정 방법 -> ~{경로 :: 지정할 fragment(~{::태그}, ~{::태그})}

                    html 파일의 경로를 지정하고 해당 html파일의 fragment를 지정하여 replace 혹은 insert

                    태그를 지정하여 인자로 넘길 수 있음
        
            fragment 인자 사용 방법 -> ${인자}

                    fragment를 지정할 때 인자로 넘긴 태그를 SpringEL 표현식을 통하여 replace 혹은 insert  

## 입력 폼

### id, name, value

    - 입력 폼과 관련하여 타임리프가 스프링과 통합되어 여러 편의 기능을 제공함

        입력 폼 편의 기능을 사용하기 위해 컨트롤러 단에서 객체를 모델을 통해 넘겨야 함
        모델로 넘어온 객체를 타임리프를 통해 th:object, th:field 지정 

```html

    <form action="item.html" th:action th:object="${item}" method="post">

        <input type="text" id="itemName" th:field="*{itemName}" class="form-control" placeholder="이름을 입력하세요">

```

        ${item.itemName} -> *{itemName} 

            *{...} 선택 변수 식을 이용하여 간소화 가능


        렌더링 시 th:field가 지정되어 있는 태그에 id, name, value 속성을 자동으로 만들어 줌
        
        value -> 객체에 들어있는 값 부여, 빈 객체의 경우 값이 없으므로 빈값으로 처리되어 들어감

### 단일 checkbox

    - 단일 체크박스의 문제점


        체크박스 체크 시에는 "on"이 대입되어 전송되나 체크박스 체크 해제 시에는 아무런 값도 전송되지 않음
        아무런 값도 전송되지 않은 것이 체크를 해제하여 그런 것인지 어떠한 행위 자체가 일어나지 않아 그런 것인지
        구분을 할 수 없어 혼란을 야기시킴

            스프링 타입 컨버터에 의해 "on" 값이 들어있는 경우 true 값으로 처리됨

        뷰 렌더링 시 체크박스에 체크된 상태로 나타내어야 하는 경우에 checked 속성을 추가하고
        체크박스에 체크를 해제된 상태로 나타내어야 하는 경우 checekd 속성 자체를 없애야하는 번거러운 과정이 존재
        

    - 스프링에서의 해결 방법

```html

    <input type="checkbox" id="open" name="open" class="form-check-input"> 
    <input type="hidden" name="_open" value="on"/>

```

        체크박스 체크 시 : open에 값이 들어있으니 _open은 무시 -> true 값으로 처리함
        체크박스 체크 해제 시 : open 속성 자체가 없고 _open만 있는 것을 확인 -> false 값으로 처리함 


    - 타임리프에서의 해결 방법

```html

        <input type="checkbox" id="open" th:field="*{open}" class="form-check-input">

```
        히든필드 자동 생성

            개발자가 직접 히든필드를 작성하여야 했던 것을 타임리프가 렌더링 시 
            th:field가 지정되어 있는 태그에 히든필드를 자동으로 만들어 줌

        checked 속성 자동 부여

            개발자가 직접 체크박스 체크, 체크해제 여부를 판단하여 checked 속성을 부여했어야 하는 것을
            타임리프가 렌더링 시 th:field가 지정되어 있는 태그에 chekced 속성을 자동으로 처리해 줌            


### 멀티 checkbox

    - 타임리프가 단일 체크박스에서 지원하는 기능을 그대로 제공함

        히든필드 자동 생성, checked 속성 자동 부여

    - 반복을 통한 멀티 체크박스 표현 시 id, label 동적 처리 문제 해결

```html

    <div th:each="region : ${regions}" class="form-check form-check-inline">
          <input type="checkbox" th:field="*{regions}" th:value="${region.key}" class="form-check-input">
          <label th:for="${#ids.prev('regions')}" th:text="${region.value}" class="form-check-label">서울</label>
    </div>

```

        타임리프가 렌더링 될 때 id는 모두 다른 값이 들어가야 하므로
        th:each로 반복되는 경우 th:field가 지정되어 Id 자동 생성 시 
        순차적으로 숫자를 뒤에 붙여 렌더링 함

        라벨의 id 역시 각각의 체크박스 input 태그에 들어간 id와 같은 값을 부여받아야 하므로
        유틸리티 객체를 이용하여 동일한 id를 붙여받을 수 있도록 함
        
### 라디오 버튼

    - checked 속성 자동 부여함

    - 반복을 통한 라디오 버튼 표현 시 id, label 동적 처리 문제 해결

    - 라디오 버튼은 히든 필드를 자동으로 생성해주지 않는데 그 이유는 한번 라디오 버튼이 체크되면 그 뒤에는 라디오 버튼을 없앨 수 없기 때문임

        최초 라디오 버튼을 클릭하지 않으면 null로 넘어가나 사용자가 라디오 버튼을 체크하지 않았다는 것이 명확하므로 혼돈의 여지가 없음

```html

<div th:each="type : ${itemTypes}" class="form-check form-check-inline">
    <input type="radio" th:field="*{itemType}" th:value="${type.name()}" class="form-check-input">
    <label th:for="${#ids.prev('itemType')}" th:text="${type.description}" class="form-check-label">
        BOOK
    </label>
</div>

```

### 셀렉트 박스

    - selected 속성 자동 부여함

```html

<select th:field="*{deliveryCode}" class="form-select"> 
    <option value="">==배송 방식 선택==</option>
    <option th:each="deliveryCode : ${deliveryCodes}" 
            th:value="${deliveryCode.code}" th:text="${deliveryCode.displayName}">
        FAST
    </option>
</select>
```