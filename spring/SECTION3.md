---
sort: 3
---

# 빈 스코프

---

## 빈 스코프 지정

~~~java

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

~~~

## 빈 스코프 종류

singleton

    - 스프링 컨테이너가 자기 자신이 종료될 때까지 빈을 관리하는 가장 넓은 범위의 스코프(디폴트 스코프)

    - 스프링 컨테이너가 항상 같은 인스턴스의 스프링 빈을 반환함

~~~java

AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SingletonBean.class);

System.out.println("after SingletonBean initialization");

SingletonBean singletonBean1 = ac.getBean(SingletonBean.class);
SingletonBean singletonBean2 = ac.getBean(SingletonBean.class);

System.out.println("singletonBean1 = " + singletonBean1);
System.out.println("singletonBean2 = " + singletonBean2);

assertThat(singletonBean1).isSameAs(singletonBean2);

ac.close();

~~~

~~~java

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

~~~

    빈 등록 시 빈을 초기화하는 과정에서 초기화 콜백 메서드가 호출되었고 이후 스프링 컨테이너에서 관리되어
    빈을 요청할 때마다 스프링 컨테이너가 같은 인스턴스를 반환했음을 알 수 있음

    SingletonBean 클래스의 초기화 콜백 메서드가 "after SingletonBean initialization" 문구 출력 이후에 호출 됨
    이는 곧 스프링 컨테이너가 빈 등록 시 빈을 초기화한 이후에 빈에 대한 요청이 있을 때마다 해당 빈의 인스턴스를 반환했음을 알 수 있음

prototype

    - 스프링 컨테이너가 빈의 생성과 의존관계 주입까지만 관여하고 더는 관리하지 않는 매우 짧은 범위의 스코프

    - 스프링 컨테이너가 항상 새로운 인스턴스의 스프링 빈을 반환함

    - @PreDestroy와 같은 종료 콜백 메서드가 호출되지 않음

~~~java

AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);

System.out.println("find prototypeBean1");
PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);

System.out.println("find prototypeBean2");
PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);

System.out.println("prototypeBean1 = " + prototypeBean1);
System.out.println("prototypeBean2 = " + prototypeBean2);

Assertions.assertThat(prototypeBean1).isNotSameAs(prototypeBean2);

ac.close();

~~~

~~~java

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

~~~

    PrototypeBean 클래스의 초기화 콜백 메서드가 "find prototypeBean1", "find prototypeBean2" 문구 출력 이후에 각각 호출 됨
    이는 곧 스프링 컨테이너가 빈 등록 시 빈을 초기화하지 않고 빈에 대한 요청이 있을 때마다 해당 빈을 초기화하여 새로운 인스턴스를 반환했음을 알 수 있음
    또한, 싱글톤 스코프와는 다르게 종료 콜백 메서드가 호출되지 않았는데 스프링 컨테이너가 빈의 초기화 이후에 해당 인스턴스를 관리하지 않음을 의미함 

    ** 프로토타입 스코프를 가지는 빈은 빈 등록 시 초기화 되지 않고 빈에 대해 요청 시마다 생성 및 초기화 됨
    ** 프로토타입 스코프를 가지는 빈은 스프링 컨테이너에서 관리되지 않기 때문에 종료 콜백 메서드가 호출되지 않음
        
## 싱글톤과 프로토타입