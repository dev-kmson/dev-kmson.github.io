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
# 예시) spring.thymeleaf.prefix = classpath:/templates/ (default) 
spring.thymeleaf.prefix = [path]

# thymeleaf 뷰 설정을 위한 파일 확장자
# 예시) spring.thymeleaf.suffix = .html (default) 
spring.thymeleaf.suffix = [ext]

# MessageSource basename 설정
# 클래스패스 하위의 [fileName].properties 파일로 설정함을 의미
# 예시) spring.messages.basename = messages (default)
spring.messages.basename= [fileName1], [fileName2]

# 쿠키를 통한 세션 유지 설정
# 해당 옵션을 설정하지 않으면 최초 세션 이용 시 URL에 jsessionid=... 으로 값이 노출 됨
# 예시) server.servlet.session.tracking-modes = cookie
server.servlet.session.tracking-modes = [mode]

# 세션 타임아웃 설정
# 클라이언트에서 서버로 요청이 온 시점을 기준으로 세션 유지할 시간을 설정함
# 60초 단위로 설정하여야 함 (60, 120, 180, ...)
# 예시) server.servlet.session.timeout = 1800 (default)
server.servlet.session.timeout = [time]

# 스프링부트 기본 오류 페이지 설정
# 오류 처리 화면이 없을 경우 스프링 whitelabel 오류 페이지 적 
# server.error.whitelabel.enabled = true (default)
server.error.whitelabel.enabled = [boolean]

# 오류 정보 설정
# View단에서 오류 정보 노출을 허용할 것인지 쿼리파라미터 존재 시 허용할 것인지 허용 안할 것인지를 설정함
# never : 허용 안함, on_param : 쿼리 파라미터 존재 시 허용, always : 항상 허용
# server.error.include-exception = false (default)
# server.error.include-message = never (default)
# server.error.include-stacktrace = never (default)
# server.error.include-binding-errors = never (default)
server.error.include-exception = [boolean]
server.error.include-message = [opt]
server.error.include-stacktrace = [opt]
server.error.include-binding-errors = [opt]

# 오류 페이지 경로 설정
# server.error.path = /error (default)
server.error.path = [path]

# multipart form
# multipart form 허용 여부
# 예시) spring.servlet.multipart.enabled = false
spring.servlet.multipart.enabled = [boolean]
# multipart form 전송 시 파일 사이즈 제한
# 파일 하나의 최대 사이즈
# 예시) spring.servlet.multipart.max-file-size = 1MB
spring.servlet.multipart.max-file-size = [size]
# 전체 파일들의 총 사이즈
# 예시) spring.servlet.multipart.max-request-size = 10MB
spring.servlet.multipart.max-request-size = [size]

# JPA Pageable 페이지 사이즈 
# 예시) spring.data.web.pageable.default-page-size = 10 (default)  
spring.data.web.pageable.default-page-size = 20
# JPA Pageable 최대 페이지 사이즈
# 예시) spring.data.web.pageable.max-page-size = 1000 (default)
spring.data.web.pageable.max-page-size = 2000

```
