---
sort: 1
---

# 스프링 프레임워크

---

## 빈

 @Bean

    - 메서드 레벨에 사용

    - 빈 이름: 메서드 이름, 빈 객체: 메서드가 반환하는 객체

    - 메서드가 반환하는 객체가 스프링 컨테이너의 빈으로 등록됨

 @Configuration  

    - 클래스 레벨에 사용

    - 빈으로 등록할 객체와 의존 주입을 직접 설정할 수 있음

    - 내부의 @Component로 인해 컴포넌트 스캔 대상

        (조작된)클래스가 스프링 컨테이너의 빈으로 등록됨

    - 바이트코드를 조작하는 CGLIB 기술을 사용하여 싱글톤 보장

        바이트코드를 조작하여 클래스를 상속받는 임의 다른 클래스 생성 후 스프링 빈으로 등록

        조작된 클래스로 인하여 @Bean이 붙은 메서드의 반환 객체가 스프링 컨테이너에 빈으로 
        존재하면 빈 반환, 존재하지 않으면 빈 등록


~~~java


@Configuration
public class AppConfig {
    
    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
    
}
~~~

        @bean이 붙은 memberRepository메서드의 반환 객체 MemoryMemberRepository를 빈에 등록하는 과정 중에 1번,
        @bean이 붙은 memberService메서드의 반환 객체 MemberServiceImpl을 빈에 등록하는 과정 중에 1번,
        @bean이 붙은 orderService메서드의 반환 객체 OrderServiceImpl을 빈에 등록하는 과정 중에 1번,
        총 3번 memberRepository메서드가 호출됨
    
        호출될 때마다 MemoryMemberRepository 인스턴스가 생성될 것으로 예상되나 
        실질적으로 생성된 인스턴스는 1개로써 싱글톤이 보장되고 있음
    
    - @Configuration으로 인해 바이트코드 조작이 일어나므로 해당 어노테이션 제거 시 싱글톤이 보장되지 않음

## 컴포넌트 스캔

 @Component

    - 클래스 레벨에 사용

    - 해당 클래스가 @ComponentScan의 대상이 되어 스프링 컨테이너의 빈으로 등록됨

    - @Configuration과 @Bean을 이용하여 빈 객체와 의존 관계를 직접 주입한 것과 달리
      의존관계 주입을 해당 클래스 내에서 해결하여야 함

 @ComponentScan

    - 클래스 레벨에 사용

    - @Component가 붙은 클래스들을 스프링 컨테이너에 빈으로 자동 등록함

        @Component를 포함하고 있는 어노테이션
            @Controller
            @Service
            @Repository
            @Configuration

    - 기본적으로 @ComponentScan이 위치한 패키지 내의 모든 클래스가 컴포넌트 스캔 대상이 되도록 설정되어있음

        스캔을 시작할 위치를 지정할 수도 있으나 보통 @ComponentScan을 프로젝트의 최상단에 위치시킴

    - @Bean을 이용하여 수동으로 빈을 등록시킨 경우와 @Component를 이용하여 @ComponentScan에 의해
      자동으로 빈을 등록시켰을 때 빈의 이름이 동일하다면 수동 빈 등록이 우선권을 가짐
    
        스프링 부트에서는 우선권 부여 없이 오류를 발생 시킴

    - 스캔 탐색 시작 위치 변경, 특정 컴포넌트 스캔 대상 제외 등 여러 설정들을 제공함 

@ComponentScan이 붙은 클래스에 @Configuration이 없어져도 @Component 대상 클래스들이 빈으로 등록이 되는가?  
@ComponentScan이 붙은 클래스에 @Configuration이 없어지면 싱글톤을 보장하지 못하는가?


    
    
