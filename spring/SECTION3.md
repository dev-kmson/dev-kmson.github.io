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

    - 스프링 컨테이너가 자기 자신이 종료될 때까지 빈을 관리하는 가장 넓은 범위의 스코프

        디폴트 스코프로써 스코프를 따로 지정하지 않으면 싱글톤의 스코프를 가짐

prototype

    - 스프링 컨테이너가 빈의 생성과 의존관계 주입까지만 관여하고 더는 관리하지 않는 매우 짧은 범위의 스코프
