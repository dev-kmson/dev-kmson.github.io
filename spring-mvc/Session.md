---
sort: 6
---

# Session

---

## Session, Cookie

    - 일정 시간동안 동일한 사용자(브라우저)로 부터 들어오는 일련의 요구를 하나의 상태로 두어 일정하게 유지시키는 기술

        servlet이 세션에 관련된 기술을 제공함

        최초 로그인 시 servlet은 세션ID를 만들어 브라우저에 전달하고, 브라우저는 전달받은 세션ID를
        쿠키 저장소에 보관하여 이 후 서버와 통신할 때마다 쿠키 저장소에 보관된 세션ID를 전달함
        servlet은 필요한 경우 브라우저로부터 넘어온 세션ID 확인한 후에 로직을 수행함

```java

        HttpSession session = request.getSession();
        session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);

```

        로그인 시 request.getSession()을 통하여 세션을 생성하고 해당 세션에 필요한 정보를 담아 
        이 후 같은 세션ID로 요청이 오면 담아둔 정보를 활용할 수 있음

        request.getSession(true) -> 세션이 있으면 기존 세션 반환, 세션이 없으면 새로운 세션을 생성하여 반환 (default)
        request.getSession(false) -> 세션이 있으면 기존 세션 반환, 세션이 없으면 null 반환

```java

HttpSession session = request.getSession(false); 
if (session != null) {
    session.invalidate();
}

```

        로그아웃 시 세션을 조회하여 세션 존재 시 session.invalidate()를 통하여 세션을 제거

```java

@GetMapping("/")
public String homeLoginV3Spring(
        @SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Member loginMember,
        Model model) {

        ...
}

```

        매핑 메서드의 파라미터로 스프링에서 제공하는 @SessionAttribute 이용 시 
        HttpServletRequest객체로부터 일일이 세션을 확인하고 세션 내의 데이터를 확인하는 등의 
        번거로운 과정을 줄일 수 있음

        @SessionAttribute의 required 옵션이 true인 경우(default) name의 이름을 가지는 세션 속성이 없으면 예외 발생
        @SessionAttribute의 name 옵션을 통해 세션에 담긴 속성을 가져올 수 있음

## TrackingModes, TimeOut

### TrackingModes

    servlet은 브라우저가 쿠키를 지원하는지 확인할 수 있는 방법이 없어 세션ID를 
    브라우저의 쿠키 저장소에 저장함과 동시에 URL에도 jsessionid라는 이름으로
    세션ID를 저장함

    일반적으로, 세션ID가 사용자 브라우저의 URL에 노출되는 것으로 좋지 않으므로
    application.properties에 server.servlet.session.tracking-modes=cookie를
    작성하여 세션을 유지하는 방법으로 쿠키를 이용함을 명시할 수 있음

### TimeOut

    사용자(브라우저)의 요청이 일어난 후에 특정 시간이 지나면 세션을 삭제시킴
    default timeout 시간을 1800초로써 세션 유지시간을 변경시키고 싶은 경우
    application.properties에 server.servlet.session.timeout=60과
    같이 작성하여 기존 1800초에서 60초로 변경할 수 있음

    해당 초는 LastAccessedTime을 기준으로 하여 시작되며 사용자(브라우저)로 부터 새로운
    요청이 오게되면 LastAccessedTime이 갱신되어 새로운 세션 유지시간을 가지게 됨

    HttpServletRequest객체로 부터 getLastAccessedTime()를 통하여 가장 최근의
    세션 접근 시간을 알아낼 수 있으며 setMaxInactiveInterval(1800)과 같이 작성하여
    특정한 행위 시에 세션 시간을 다르게 가지도록 할 수도 있음