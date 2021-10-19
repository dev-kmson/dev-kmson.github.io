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

singleton

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

```java

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

prototype

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

```java

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
        
## 프로토타입의 문제점

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

## Provider

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

```java

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

```java

PrototypeBean.init com.devson.springtest.scope.SingletonWithPrototypeTest1$PrototypeBean@221a3fa4 
PrototypeBean.init com.devson.springtest.scope.SingletonWithPrototypeTest1$PrototypeBean@2438dcd

```

    스프링에서 제공하는 ObjectProvider 대신 자바 표준 javax.inject.Provider를 사용하며
    자바 표준 javax.injectProvider에서 제공하는 get()를 호출한다는 점을 제외하고는 ObjectProvider를
    사용한 테스트 케이스와 동일

## Provider 선택 기준

    ObjectProvider는 DL을 위한 많은 편의 기능을 제공하고 있으며 
    스프링 외에 별도의 라이브러리가 필요 없기 때문에 사용하기 편리함

    일반적으로 ObjectProvider를 이용하되 ObjectProvider는 스프링에 의존적이므로 
    스프링이 아닌 다른 컨테이너에서 사용할 수 있어야 한다면 자바 표준 Provider를 이용
    