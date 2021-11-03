---
sort: 1
---

# ThymeLeaf

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
