---
sort: 5
---

# 스프링부트 Properties 설정

---

```properties

# 스프링부트 내장 톰캣 포트 변경
# 예시) server.port = 8080
server.port = [port]

# HTTP 1.1에 대한 로깅 레벨 지정
# debug, error, fatal, info, off, trace, warn 
# 예시) logging.level.org.apache.coyote.http11 = debug 
logging.level.org.apache.coyote.http11 = [level]

# JSP 뷰 설정을 위한 파일 경로
# 예시) spring.mvc.view.prefix = /WEB-INF/views/
spring.mvc.view.prefix = [path]

# JSP 뷰 설정을 위한 파일 확장자
# 예시) spring.mvc.view.suffix = .jsp
spring.mvc.view.suffix = [ext]

```
