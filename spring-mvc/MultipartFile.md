---
sort: 12
---

# MultipartFile

    일반적으로 form이 전송될 때는 데이터가 문자 형태로 서버에 넘어오게 됨
    그러나, 파일이 첨부될 경우 문자가 아닌 바이너리로 전송이 되어야 하므로
    문자와 바이너리를 동시에 전송해야 하는 상황이 발생함

    이러한 문제를 해결하기 위해 HTTP에서는 multipart/form-data라는 
    전송 방식을 제공함

```html

    <form th:action method="post" enctype="multipart/form-data">
        <ul>
            <li>상품명 <input type="text" name="itemName"></li>
            <li>파일<input type="file" name="file" ></li> 
        </ul>
        <input type="submit"/>
    </form>

```

```text
    
    Content-Type: multipart/form-data; boundary=----xxxx
    
    ------xxxx
    Content-Disposition: form-data; name="itemName"
    Spring
      
    ------xxxx
    Content-Disposition: form-data; name="file"; filename="test.data"
    Content-Type: application/octet-stream
    sdklajkljdf...

```

    일반적인 HTTP 헤더 바디와는 다르게 바디 영역에 또 다른 헤더와 바디로 구성되어 있음 

    spring.servlet.multipart.enabled 옵션이 false로 설정되어있지 않다면
    DispatcherServlet에서 MultipartResolver를 호출,
    MultipartResolver가 multipart 요청인 경우 서블릿 컨테이너가 전달하는 
    일반적인 HttpServletRequest를 MultipartHttpServletRequest로 변환하여 반환함

        정확히는 MultipartHttpServletRequest 인터페이스를 
        구현한 StandardMultipartHttpServletRequest를 반환

    multipart 요청이라고하여도 직접적으로 MultipartHttpServletRequest를 이용하지는 않는데
    이는 더 편리한 MultipartFile을 스프링에서 제공하기 때문임

```java

    Collection<Part> parts = request.getParts();
    for (Part part : parts) {
        if (StringUtils.hasText(part.getSubmittedFileName())) {
            String fullPath = fileDir + part.getSubmittedFileName();
            part.write(fullPath);
        }   
    }
```

    multipart 형식은 전송 데이터를 Part로 나누어 전송하고 Parts에 각각의 Part들이 담기게 됨
    서블릿이 제공하는 Part에는 multipart 형식을 편리하게 읽을 수 있는 다양한 메서드를 제공함

    part.getSubmittedFileName() : 클라이언트가 전송한 파일명
    part.getInputStream() : Part의 전송 데티어를 읽음
    Part.write(...) : Part를 통해 전송된 데이터를 저장

    서블릿이 제공하는 Part를 이용하여 편리하게 사용할 수는 있으나 
    form으로 전송된 데이터를 받기 위해 request를 이용하여야 하고
    request의 Part에서 파일 관련 처리 부분을 별도로 작성하여야 
    하는 등 불편함이 존재함
    
```java

    @PostMapping("/upload")
    public String saveFile(@RequestParam String itemName,
                        @RequestParam MultipartFile file, 
                        HttpServletRequest request) throws IOException{
    
        if (!file.isEmpty()) {
            String fullPath=fileDir+file.getOriginalFilename();
            file.transferTo(new File(fullPath));
        }
             
             //...
    }

```

    스프링은 MultipartFile이라는 인터페이스를 통하여 multipart로 
    전송된 파일을 편리하게 읽고 쓸 수 있도록 지원함

    request에서 form으로 전송된 데이터를 추출 및 파일 처리를 하지 않고
    스프링이 제공하는 MultipartFile을 이용하여 파일 관련 처리를 분리할 수 있게 됨

    file.getOriginalFilename() : 클라이언트가 전송한 파일명
    file.transferTo(...) : 파일 저장