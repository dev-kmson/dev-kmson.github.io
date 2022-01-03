---
sort: 11
---

# Converter

## Converter

    객체를 문자로 변환, 문자를 객체로 변환 등 타입을 변환할 때 이용되는 스프링이 제공하는 인터페이스

```java

    public interface Converter<S, T> {
        T convert(S source);
    }

```

    해당 인터페이스를 구현하여 S를 T로 변환하는 로직을 작성함

```java

    @Slf4j
    public class StringToIpPortConverter implements Converter<String, IpPort> {
        
        @Override
        public IpPort convert(String source) {
            
            log.info("convert source={}", source);
            
            String[] split = source.split(":");
            String ip = split[0];
            int port = Integer.parseInt(split[1]);
            
            return new IpPort(ip, port);
            
        }
        
    }

```

    타입 변환이 필요할 때마다 타입에 맞는 Converter를 구현하고
    해당 클래스를 인스턴스화하여 사용하기에는 매우 불편함

    각 Converter들을 모아서 편리하게 사용할 수 있는 ConversionService를 스프링이 제공함

```java

    public interface ConversionService {
        
        boolean canConvert(@Nullable Class<?> sourceType, Class<?> targetType);
    
        boolean canConvert(@Nullable TypeDescriptor sourceType, TypeDescriptor targetType);
    
        <T> T convert(@Nullable Object source, Class<T> targetType);
    
        Object convert(@Nullable Object source, @Nullable TypeDescriptor sourceType, TypeDescriptor targetType);
    
    }

```

```java

    @Test
    void conversionService(){
    
        DefaultConversionService conversionService = new DefaultConversionService();
        conversionService.addConverter(new StringToIpPortConverter());

        IpPort ipPort = conversionService.convert("127.0.0.1:8080", IpPort.class);
        assertThat(ipPort).isEqualTo(new IpPort("127.0.0.1", 8080));
            
    }

```

    타입 변환이 필요한 경우 ConversionService의 구현체인 DefaultConversionService를 이용하여
    addConverter()를 통한 Converter 추가 및 convert()를 이용하여 타입 변환을 할 수 있음

    ConversionService 인터페이스에는 Converter를 추가하는 addConverter()이 존재하지 않는데
    이는 DefaultConversionService 구현체가 타입을 변환하는데 이용하는 ConverionService 인터페이스뿐만
    아니라 Converter를 등록하는 ConverterRegistry 인터페이스도 구현했기 때문에
    DefaultConversionService 구현체를 이용하면 Converter의 등록과 사용이 모두 가능함

    참고로, 하나의 인터페이스에 여러 역할이 존재하지 않도록 Converter의 사용과 등록하는 역할을 분리한 것은
    인터페이스 분리 원칙(Interface Segregation Principle)이 적용된 것임

[인터페이스 분리 원칙, Interface Segregation Principle](https://ko.wikipedia.org/wiki/%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4_%EB%B6%84%EB%A6%AC_%EC%9B%90%EC%B9%99)

```java

    @Configuration
    public class WebConfig implements WebMvcConfigurer {
        
        @Override
        public void addFormatters(FormatterRegistry registry) {
            registry.addConverter(new StringToIpPortConverter());
        }
        
    }

```

    웹 어플리케이션에 Converter를 등록하기 위해선 WebMvcConfigurer를 
    구현한 WebConfig에 addFormatters()를 Override하여야 함

    Converter를 등록시키면 웹 호출 시에 등록한 Converter가 작동하는 것을 확인할 수 있음

    DispatcherServlet에서 매핑된 메서드의 인자를 처리하기 위한 ArgumentResolver를 호출, 
    ArgumentResolver가 ConversionService를 이용하여 타입을 변환함

```html
    
    <!DOCTYPE html>
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body> 
    <ul>
        <li>${number}: <span th:text="${number}" ></span></li>
        <li>${{number}}: <span th:text="${{number}}" ></span></li>
        <li>${ipPort}: <span th:text="${ipPort}" ></span></li>
        <li>${{ipPort}}: <span th:text="${{ipPort}}" ></span></li>
    </ul>
    </body>

```

    ${{...}} 표현식을 사용하면 자동으로 ConversionService가 적용된 결과 값을 출력하여 보여줌
    일반 변수 표현식인 ${...} 사용 시 ConversionService가 적용되지 않은 결과 값을 출력하여 보여줌 


```html

    <!DOCTYPE html>
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body>
    <form th:object="${form}" th:method="post">
        th:field <input type="text" th:field="*{ipPort}"><br/>
        th:value <input type="text" th:value="*{ipPort}">(보여주기 용도)<br/> 
        <input type="submit"/>
    </form>
    </body>
    </html>

```
    
    th:field 사용 시 ConversionService가 적용된 결과 값을 출력하여 보여줌
    th:value 사용 시 ConversionService가 적용되지 않은 결과 값을 출력하여 보여줌
    
## Formatter

    일반적으로 타입 변환 시에 가장 많이 이용되는 객체를 문자로 변경 혹은 문자를 객체로 변경하는 기능을 제공

```java

    public interface Printer<T> {
        String print(T object, Locale locale);
    }
    
    public interface Parser<T> {
        T parse(String text, Locale locale) throws ParseException;
    }
    
    public interface Formatter<T> extends Printer<T>, Parser<T> {}

```

    객체를 문자로 변경하는 Printer와 문자를 객체로 변경하는 Parser 인터페이스를 상속받음

    각 나라 별로 문자를 표현하는 방법이 다르므로 locale을 받아서 
    해당하는 locale에 맞는 형식으로 변환할 수 있도록 함

```java
    
    @Slf4j
    public class MyNumberFormatter implements Formatter<Number> {
        
        @Override
        public Number parse(String text, Locale locale) throws ParseException {
            log.info("text={}, locale={}", text, locale);
            NumberFormat format = NumberFormat.getInstance(locale);
            return format.parse(text);
        }
        
        @Override
        public String print(Number object, Locale locale) {
            log.info("object={}, locale={}", object, locale);
            return NumberFormat.getInstance(locale).format(object);
        }
        
    }

```

```java

    class MyNumberFormatterTest {
    
        MyNumberFormatter formatter = new MyNumberFormatter();
        
        @Test
        void parse() throws ParseException {
            Number result = formatter.parse("1,000", Locale.KOREA); 
            assertThat(result).isEqualTo(1000L);
        }
        
        @Test
        void print() {
            String result = formatter.print(1000, Locale.KOREA);
            assertThat(result).isEqualTo("1,000");
        }
    }
    
```

    formatter를 구현하여 문자를 객체로 변경하거나 객체를 문자로 변경할 수 있도록 하였고
    NumberFormat을 이용하여 숫자에 쉼표를 출력하거나 쉼표가 포함된 문자를
    숫자로 변경할 수 있도록 하였음

```java

    public class FormattingConversionServiceTest {
        @Test
        void formattingConversionService() {
            
            DefaultFormattingConversionService conversionService = 
                    new DefaultFormattingConversionService();
            
            //컨버터 등록
            conversionService.addConverter(new StringToIpPortConverter());
            conversionService.addConverter(new IpPortToStringConverter()); 
            
            //포맷터 등록
            conversionService.addFormatter(new MyNumberFormatter());
    
            //컨버터 사용
            IpPort ipPort = conversionService.convert("127.0.0.1:8080", IpPort.class);
            assertThat(ipPort).isEqualTo(new IpPort("127.0.0.1", 8080)); 
            
            //포맷터 사용
            assertThat(conversionService.convert(1000, String.class)).isEqualTo("1,000");
            assertThat(conversionService.convert("1,000", Long.class)).isEqualTo(1000L);
          }
    }

```    

    일반적인 ConversionService는 Converter만을 등록할 수 있고 Formatter는 등록할 수 없으나
    FormattingConersionService는 Formatter 등록도 지원하는 ConversionService임

    DefaultFormattingConversionService는 Formatter를 지원하는 ConversionService에
    기본적인 통화, 숫자 관련 몇가지 기본 Formatter를 추가해서 제공함

    스프링에서는 DefaultFormattingConversionService를 상속받은 WebConversionService를 내부에서 사용함

```java

    @Configuration
    public class WebConfig implements WebMvcConfigurer {
        
        @Override
        public void addFormatters(FormatterRegistry registry) {
            registry.addConverter(new StringToIpPortConverter());
            registry.addFormatter(new MyNumberFormatter());
        }
        
    }
      

```

    웹 어플리케이션에 Formatter를 등록하기 위해선 Converter와 마찬가지로
    WebMvcConfigurer를 구현한 WebConfig에 addFormatters()를 Override하여야 함

    스프링은 수많은 날짜나 시간 관련 Formatter를 제공하고 있음
    다만, ArgumentResolver에 의해 타입 변환이 이루어질 때 
    각 필드별로 다른 형식으로 Format을 지정하기 어려움

    이를 위해 애노테이션 기반으로 형식을 지정하여 사용할 수 있는
    @NumberFormat과 @DateTimeFormat을 제공함

        @NumberFormat : 숫자 관련 형식 지정 Formatter (NumberFormatAnnotationFormatterFactory)
        @DateTimeFormat : 날짜 관련 형식 지정 Formatter (Jsr310DateTimeFormatAnnotationFormatterFactory)

[Format 공식문서](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#format)  
[Format 애노테이션](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#format-CustomFormatAnnotations)

```java

@Data
static class Form {
    
    @NumberFormat(pattern = "###,###")
    private Integer number;
    
    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime localDateTime;
    
}

```

    웹 어플리케이션 기동 시 스프링이 타입 변환 시에 ConversionService를 이용하지만
    이는 어디까지나 View와 관련된 ArgumentReslover가 관여할 때의 이야기임

    HTTP 바디의 내용을 읽어 객체로 변환되거나 객체를 HTTP 바디에 입력하는
    HttpMessageConverter가 사용될 때는 ConversionService가 적용되지 않음
    Json 데이터를 처리하기 위해 HttpMessageConverter는 내부적으로 Jackson
    라이브러리를 사용하며 이 때, 특정한 Format으로의 변환이 필요하다면
    전적으로 해당 라이브러리에서 도움을 받아야 함

    