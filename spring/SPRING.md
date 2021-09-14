---
sort: 1
---

# 스프링 프레임워크

---

@Bean

    - 메서드 레벨에 사용
    - 빈 이름: 메서드 이름, 빈 객체: 메서드가 반환하는 객체
    - 메서드가 반환하는 객체가 스프링 컨테이너의 빈으로 등록됨

@Configuration  

    - 클래스 레벨에 사용
    - 내부의 @Component로 인해 컴포넌트 스캔 대상
    - (조작된)클래스가 스프링 컨테이너의 빈으로 등록됨
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

@ComponentScan

    - @Component가 붙은 모든 클래스를 스프링 빈으로 등록

    
    
