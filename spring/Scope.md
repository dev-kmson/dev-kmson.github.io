---
sort: 3
---

# 빈 스코프

---

## 빈 스코프 지정

```java

// @ComponentScan과 @Component를 이용한 자동 빈 등록에서의 빈 스코프 지정
@Scope("prototype")
@Component
public class HelloBean {
    
}

// @Configuration과 @Bean을 이용한 수동 빈 등록에서의 빈 스코프 지정
@Scope("prototype")
@Bean
PrototypeBean helloBean() {
    return new helloBean();
}

```

## 빈 스코프 종류

### singleton

    - 스프링 컨테이너가 자기 자신이 종료될 때까지 빈을 관리하는 가장 넓은 범위의 스코프(디폴트 스코프)

    - 스프링 컨테이너가 항상 같은 인스턴스의 스프링 빈을 반환함

```java

AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SingletonBean.class);

System.out.println("after SingletonBean initialization");

SingletonBean singletonBean1 = ac.getBean(SingletonBean.class);
SingletonBean singletonBean2 = ac.getBean(SingletonBean.class);

System.out.println("singletonBean1 = " + singletonBean1);
System.out.println("singletonBean2 = " + singletonBean2);

assertThat(singletonBean1).isSameAs(singletonBean2);

ac.close();

```

```text

17:57:54.169 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@318ba8c8
17:57:54.184 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalConfigurationAnnotationProcessor'
17:57:54.220 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerProcessor'
17:57:54.221 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerFactory'
17:57:54.223 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalAutowiredAnnotationProcessor'
17:57:54.224 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalCommonAnnotationProcessor'
17:57:54.231 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'singletonTest.SingletonBean'
SingletonBean.init
after SingletonBean initialization
singletonBean1 = com.devson.springtest.scope.SingletonTest$SingletonBean@644baf4a
singletonBean2 = com.devson.springtest.scope.SingletonTest$SingletonBean@644baf4a
SingletonBean.destroy

```

    SingletonBean 클래스의 초기화 콜백 메서드가 "after SingletonBean initialization" 문구 출력 이전에 호출 됨
    이는 곧 스프링 컨테이너가 빈 등록 시 빈을 초기화한 이후에 빈에 대한 요청이 있을 때마다 해당 빈의 인스턴스를 반환했음을 알 수 있음

### prototype

    - 스프링 컨테이너가 빈의 생성과 의존관계 주입까지만 관여하고 더는 관리하지 않는 매우 짧은 범위의 스코프

    - 스프링 컨테이너가 항상 새로운 인스턴스의 스프링 빈을 반환함

    - @PreDestroy와 같은 종료 콜백 메서드가 호출되지 않음

```java

AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);

System.out.println("find prototypeBean1");
PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);

System.out.println("find prototypeBean2");
PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);

System.out.println("prototypeBean1 = " + prototypeBean1);
System.out.println("prototypeBean2 = " + prototypeBean2);

Assertions.assertThat(prototypeBean1).isNotSameAs(prototypeBean2);

ac.close();

```

```text

17:48:03.127 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@318ba8c8
17:48:03.140 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalConfigurationAnnotationProcessor'
17:48:03.163 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerProcessor'
17:48:03.165 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.event.internalEventListenerFactory'
17:48:03.167 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalAutowiredAnnotationProcessor'
17:48:03.168 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'org.springframework.context.annotation.internalCommonAnnotationProcessor'
find prototypeBean1
PrototypeBean.init
find prototypeBean2
PrototypeBean.init
prototypeBean1 = com.devson.springtest.scope.PrototypeTest$PrototypeBean@644baf4a
prototypeBean2 = com.devson.springtest.scope.PrototypeTest$PrototypeBean@7526515b

```

    PrototypeBean 클래스의 초기화 콜백 메서드가 "find prototypeBean1", "find prototypeBean2" 문구 출력 이후에 각각 호출 됨
    이는 곧 스프링 컨테이너가 빈 등록 시 빈을 초기화하지 않고 빈에 대한 요청이 있을 때마다 해당 빈을 초기화하여 새로운 인스턴스를 반환했음을 알 수 있음
    또한, 싱글톤 스코프와는 다르게 종료 콜백 메서드가 호출되지 않았는데 스프링 컨테이너가 빈의 초기화 이후에 해당 인스턴스를 관리하지 않음을 의미함 

    ** 프로토타입 스코프를 가지는 빈은 빈 등록 시 초기화 되지 않고 빈에 대해 요청 시마다 생성 및 초기화 됨
    ** 프로토타입 스코프를 가지는 빈은 스프링 컨테이너에서 관리되지 않기 때문에 종료 콜백 메서드가 호출되지 않음
        
#### prototype 스코프의 문제점

    프로토타입 스코프를 가지는 빈은 스프링 컨테이너에 요청이 올 때마다 새로 인스턴스를 생성하여 반환함
    그러나 싱글톤 스코프로 관리되는 빈에서 프로토타입 스코프를 가진 빈을 의존관계로 주입 받는 경우에
    싱글톤 스코프를 가진 빈이 해당 인스턴스를 계속 참조하게 되므로 마치 프로토타입이 아닌 싱글톤으로 관리되는 것처럼 보이게 됨

```java

AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SingletonBean.class, PrototypeBean.class);

SingletonBean singletonBean1 = ac.getBean(SingletonBean.class);
int count1 = singletonBean1.logic();
assertThat(count1).isEqualTo(1);

SingletonBean singletonBean2 = ac.getBean(SingletonBean.class);
int count2 = singletonBean2.logic();
assertThat(count2).isEqualTo(2);

@Scope("singleton")
static class SingletonBean {

    private final PrototypeBean prototypeBean;

    public SingletonBean(PrototypeBean prototypeBean) {
        this.prototypeBean = prototypeBean;
    }

    public int logic() {
        prototypeBean.addCount();
        return prototypeBean.getCount();
    }
}

```

    싱글톤 스코프를 가진 빈의 singletonBean1과 singletonBean2는 모두 같은 인스턴스이므로 
    SingletonBean의 멤버 변수인 prototypeBean 또한 같은 인스턴스를 참조함
    따라서, PrototypeBean의 멤버 변수인 count를 증가시키면 같은 인스턴스를 참조하므로 값이 누적됨

    위의 문제를 해결하기 위해 스프링 컨테이너를 통해 필요한 의존관계를 주입받지 않고
    스프링 컨테이너를 통해 필요한 의존관계를 찾는 DL 기능이 필요함

#### 해결 방안, Provider

    - DL 기능을 이용하여 지정한 빈을 스프링 컨테이너에서 대신 찾아주는 역할을 함

        * DI : Dependency Injection, 의존관계 주입
        * DL : Dependency Lookup, 의존관계 조회(탐색)

ObjectProvider
    
    - 스프링에서 제공하는 Provider

    - getObject()를 통하여 필요로 하는 빈을 얻어옴
    
    - 과거에는 ObjectFactory를 사용하였으나 현재는 여러 편의 기능이 추가된 ObjectProvider를 사용

    - 스프링에 의존적    

```java

AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SingletonBean.class, PrototypeBean.class);

SingletonBean singletonBean1 = ac.getBean(SingletonBean.class);
int count1 = singletonBean1.logic();
assertThat(count1).isEqualTo(1);

SingletonBean singletonBean2 = ac.getBean(SingletonBean.class);
int count2 = singletonBean2.logic();
assertThat(count2).isEqualTo(1);

@Scope("singleton")
static class SingletonBean {

    private final ObjectProvider<PrototypeBean> prototypeBeanProvider;

    public SingletonBean(ObjectProvider<PrototypeBean> prototypeBeanProvider) {
        this.prototypeBeanProvider = prototypeBeanProvider;
    }

    public int logic() {
        PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
        prototypeBean.addCount();
        return prototypeBean.getCount();
    }
}

```

```text

PrototypeBean.init com.devson.springtest.scope.SingletonWithPrototypeTest1$PrototypeBean@710c2b53
PrototypeBean.init com.devson.springtest.scope.SingletonWithPrototypeTest1$PrototypeBean@13526e59

```

    logic()가 호출되어 prototypeBean이 필요할 때 prototypeBeanProvider.getObject()를 호출하여
    스프링 컨테이너에서 빈 prototypeBean을 생성 및 초기화하여 반환받음 
    
    싱글톤 스코프를 가진 빈 SingletonBean이 등록될 때 필요로하는 의존관계 PrototypeBean을 주입 받으면
    프토토타입 스코프임에도 불구하고 항상 같은 인스턴스를 참조하게 되어 count값이 누적되는 것을 방지할 수 없으나
    SingletonBean이 등록될 때가 아닌 클라이언트가 SingletonBean의 특정 메서드를 사용할 때 ObjectProvider를 
    이용하여 프로토타입 스코프를 가진 빈 PrototypeBean을 스프링 컨테이너가 찾아서 반환하게 하면 프토토타입 스코프이기 때문에
    매번 새로 생성된 인스턴스가 반환되므로 count값이 누적되는 것을 방지할 수 있음
    
    인스턴스 출력 결과를 보면 생성된 프로토타입 스코프를 가진 PrototypeBean 인스턴스들이 서로 다른 인스턴스이고
    테스트 결과 실제 count의 값이 누적되지 않았음을 확인할 수 있음

JSR-330 Provider

    - JSR-330 자바 표준인 javax.inject.Provider

    - get()를 통하여 필요로 하는 빈을 얻어옴

    - javax.inject:javax.inject:1 라이브러리를 추가해야 사용 가능

    - 자바 표준이므로 스프링이 아닌 다른 컨테이너에서도 사용 가능

```java

AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SingletonBean.class, PrototypeBean.class);

SingletonBean singletonBean1 = ac.getBean(SingletonBean.class);
int count1 = singletonBean1.logic();
assertThat(count1).isEqualTo(1);

SingletonBean singletonBean2 = ac.getBean(SingletonBean.class);
int count2 = singletonBean2.logic();
assertThat(count2).isEqualTo(1);

@Scope("singleton")
static class SingletonBean {

    private final Provider<PrototypeBean> prototypeBeanProvider;

    public SingletonBean(Provider<PrototypeBean> prototypeBeanProvider) {
        this.prototypeBeanProvider = prototypeBeanProvider;
    }

    public int logic() {
        PrototypeBean prototypeBean = prototypeBeanProvider.get();
        prototypeBean.addCount();
        return prototypeBean.getCount();
    }
}

```

```text

PrototypeBean.init com.devson.springtest.scope.SingletonWithPrototypeTest1$PrototypeBean@221a3fa4 
PrototypeBean.init com.devson.springtest.scope.SingletonWithPrototypeTest1$PrototypeBean@2438dcd

```

    스프링에서 제공하는 ObjectProvider 대신 자바 표준 javax.inject.Provider를 사용하며
    자바 표준 javax.injectProvider에서 제공하는 get()를 호출한다는 점을 제외하고는 ObjectProvider를
    사용한 테스트 케이스와 동일

#### Provider 선택 기준

    ObjectProvider는 DL을 위한 많은 편의 기능을 제공하고 있으며 
    스프링 외에 별도의 라이브러리가 필요 없기 때문에 사용하기 편리함

    일반적으로 ObjectProvider를 이용하되 ObjectProvider는 스프링에 의존적이므로 
    스프링이 아닌 다른 컨테이너에서 사용할 수 있어야 한다면 자바 표준 Provider를 이용

### request    

    - HTTP 요청 하나가 들어오고 나갈 때까지 유지되는 스코프

    - 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고 관리됨

```java

package com.devson.springtest.common;

import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
import java.util.UUID;

@Component
@Scope("request")
public class MyLogger {

    private String uuid;
    private String requestURL;

    public void setRequestURL(String requestURL) {
        this.requestURL = requestURL;
    }

    public void log(String message) {
        System.out.println("[" + uuid + "][" + requestURL + "] " + message);
    }

    @PostConstruct
    public void init() {
        uuid = UUID.randomUUID().toString();
        System.out.println("[" + uuid + "] request scope bean create: " + this);
    }

    @PreDestroy
    public void close() {
        System.out.println("[" + uuid + "] request scope bean close: " + this);
    }
}

```

```java

package com.devson.springtest.web;

import com.devson.springtest.common.MyLogger;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.ObjectProvider;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;

@Controller
@RequiredArgsConstructor
public class LogDemoController {

    private final LogDemoService logDemoService;
    private final ObjectProvider<MyLogger> myLoggerProvider;

    @RequestMapping("log-demo")
    @ResponseBody
    public String logDemo(HttpServletRequest request) {
        String requestURL = request.getRequestURL().toString();
        MyLogger myLogger = myLoggerProvider.getObject();
        myLogger.setRequestURL(requestURL);

        myLogger.log("controller test");
        logDemoService.logic("testId");
        return "OK";
    }
}

```

```java

package com.devson.springtest.web;

import com.devson.springtest.common.MyLogger;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.ObjectProvider;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class LogDemoService {

    private final ObjectProvider<MyLogger> myLoggerProvider;

    public void logic(String id) {
        MyLogger myLogger = myLoggerProvider.getObject();
        myLogger.log("service id = " + id);
    }
}

```

```text

[281ba17a-73b7-49df-9874-838a749ab62d] request scope bean create: com.devson.springtest.common.MyLogger@63952645
Controller Bean myLogger = com.devson.springtest.common.MyLogger@63952645
[281ba17a-73b7-49df-9874-838a749ab62d][http://localhost:8080/log-demo] controller test
Service Bean myLogger = com.devson.springtest.common.MyLogger@63952645
[281ba17a-73b7-49df-9874-838a749ab62d][http://localhost:8080/log-demo] service id = testId
[281ba17a-73b7-49df-9874-838a749ab62d] request scope bean close: com.devson.springtest.common.MyLogger@63952645
[acc479ca-635e-4db0-bd5d-f0b63c2673fc] request scope bean create: com.devson.springtest.common.MyLogger@2ee074d9
Controller Bean myLogger = com.devson.springtest.common.MyLogger@2ee074d9
[acc479ca-635e-4db0-bd5d-f0b63c2673fc][http://localhost:8080/log-demo] controller test
Service Bean myLogger = com.devson.springtest.common.MyLogger@2ee074d9
[acc479ca-635e-4db0-bd5d-f0b63c2673fc][http://localhost:8080/log-demo] service id = testId
[acc479ca-635e-4db0-bd5d-f0b63c2673fc] request scope bean close: com.devson.springtest.common.MyLogger@2ee074d9

```

    request 스코프는 HTTP의 요청으로부터 반환까지 유지되므로 Controller의 log-demo URL 요청이 
    return을 만나 반환되는 시점에 스프링 컨테이너가 해당 빈을 관리 대상에서 제외함

    request 스코프를 가지는 빈은 스프링 컨테이너가 동일한 HTTP에 의한 빈 요청임을 구분하여 빈을 반환해 줌
    ObjectProvider를 통하여 LogDemoController와 LogDemoService에서 각각 개별적으로 빈 myLogger를 얻어왔음에도
    같은 인스턴스를 반환했음을 보아 같은 HTTP 요청임을 구분하였음을 알 수 있음

    Provider를 이용한 이유는 서버 구동 시 스프링 컨테이너가 빈에 대한 의존관계 주입 과정 중
    LogDemoController와 LogDemoService 클래스에서 빈 myLogger에 대한 의존관계 주입이 필요하기 때문에 주입 시도 함
    이 때, 빈 myLogger는 request 스코프를 가지므로 서버 구동 시점에서는 해당 빈을 찾을 수 없음
    따라서, Provider를 이용하여 적절한 시점에 빈을 반환받도록 함

#### proxyMode

    - @Scope 옵션을 통한 의존관계 주입 지연

        서버 구동 시 request 의존관계 주입 문제로 인하여 Provider를 이용하여 의존관계 주입 시점을 지연시켜 해결하였으나
        Provider를 이용하지 않고 @Scope의 옵션을 통하여 해결할 수도 있음 

```java

@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)

```

    proxyMode 옵션 설정 시 빈이 등록되는 시점에 CGLIB 기술을 이용하여 실제 클래스를 상속 받는 
    가짜 클래스를 만들어 빈으로 등록시키고 의존관계 주입 시점에서 빈으로 등록된 가짜 프록시 객체를 의존관계로 주입함
    이 후, 실제 빈을 필요로 하는 요청이 오면 가짜 프록시 객체의 내부에서 실제 빈을 요청하는 위임 로직을 통하여 실제 빈을 반환함
    
    가짜 프록시 객체는 실제 request 스코프와는 별개로 싱글톤 스코프처럼 동작하며 실제 빈 요청 시 위임 로직을 통하여 실제 빈을 반환하는 역할만 함

#### 의문사항
    
    prototype 스코프는 빈을 필요로 하는 시점에 새로운 인스턴스를 생성해서 반환한다
    request 스코프는 HTTP 요청이 올 때마다 스프링이 내부적으로 빈을 생성하고 빈을 필요로 할 때 해당 인스턴스를 반환하는가
    아니면 HTTP 요청과는 무관하게 빈을 필요로 할 때 내부적으로 빈을 생성하고 해당 인스턴스를 반환하는가
    만약 후자라면, 
    HTTP 요청의 구분은 어떻게 처리하는지? 
    정확히 HTTP 요청 시점에서부터 시작하는 스코프가 아니게 됨?


### session

### application

### websocket