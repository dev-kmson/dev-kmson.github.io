---
sort: 1
---

# properties 설정

---

```properties

# 스프링부트 내장 톰캣 포트 변경
# 예시) server.port = 8080
server.port = [port]

# HTTP 1.1에 대한 로깅 레벨 지정
# debug, error, fatal, info, off, trace, warn 
# 예시) logging.level.org.apache.coyote.http11 = debug 
logging.level.org.apache.coyote.http11 = [level]

# 전체 로깅 레벨 지정
# 예시) logging.level.root = info (default)
logging.level.root = [level]

# 특정 패키지 하위의 로깅 레벨 지정
# 위의 HTTP 1.1에 대한 로깅 레벨 지정도 실은 패키지의 로깅 레벨을 지정한 것
# debug, error, fatal, info, off, trace, warn
# 예시) logging.level.com.devson = info
logging.level.com.devson = [level]

# JSP 뷰 설정을 위한 파일 경로
# 예시) spring.mvc.view.prefix = /WEB-INF/views/
spring.mvc.view.prefix = [path]

# JSP 뷰 설정을 위한 파일 확장자
# 예시) spring.mvc.view.suffix = .jsp
spring.mvc.view.suffix = [ext]

# thymeleaf 뷰 설정을 위한 파일 경로
# 예시) spring.thymeleaf.prefix=classpath:/templates/ (default) 
spring.thymeleaf.prefix = [path]

# thymeleaf 뷰 설정을 위한 파일 확장자
# 예시) spring.thymeleaf.suffix=.html (default) 
spring.thymeleaf.suffix = [ext]

# MessageSource basename 설정
# 클래스패스 하위의 [fileName].properties 파일로 설정함을 의미
# 예시) spring.messages.basename=messages (default)
spring.messages.basename= [fileName1], [fileName2]

```
