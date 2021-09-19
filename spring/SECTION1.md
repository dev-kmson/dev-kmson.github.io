---
sort: 1
---

# 빈 등록

---

## 수동 빈 등록

@ Bean

    - 메서드 레벨에 사용

    - 빈 이름: 메서드 이름, 빈 객체: 메서드가 반환하는 객체

    - 메서드가 반환하는 객체가 스프링 컨테이너의 빈으로 등록됨

@ Configuration  

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

## 자동 빈 등록

@ Component

    - 클래스 레벨에 사용

    - 해당 클래스가 @ComponentScan의 대상이 되어 스프링 컨테이너의 빈으로 등록됨

    - @Configuration과 @Bean을 이용하여 빈 객체와 의존 관계를 직접 주입한 것과 달리
      의존관계 주입을 해당 클래스 내에서 해결하여야 함

@ ComponentScan

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

@ Autowired

    - 메서드, 필드 레벨에 사용
    
    - 스프링 컨테이너가 해당 메서드의 인자 혹은 필드 변수에 인스턴스를 자동으로 주입
    
        Map 또는 List로 스프링 빈에 등록된 같은 타입의 구현체들을 전부 받아올 수도 있음

~~~java

public class DiscountService {
    private final Map<String, DiscountPolicy> policyMap;
    private final List<DiscountPolicy> policies;

    public DiscountService(Map<String, DiscountPolicy> policyMap, List<DiscountPolicy> policies) {
        this.policyMap = policyMap;
        this.policies = policies;
    }    
}
~~~

        스프링 컨테이너가 DiscountPolicy 타입의 구현체들을 알아서 주입해줌

    - 의존관계 주입 방법 (일반적으로 생성자 주입을 사용)
    
        생성자 주입 :

            의존관계를 주입 받아야 할 변수들이 생성자를 통해 명시되므로 의존관계 필요성 파악 용이

            생성자를 통해 주입 받으므로 필드 변수에 final 사용 가능하므로 Immutable함

            @RequiredArgsConstructor(롬복 라이브러리를 통해 사용자가 직접 생성자 생성하지 않아도 의존관계 주입 가능)
        
        수정자 주입 :

            의존관계 주입에 대한 필요성이 명시되지 않음

            수정자 메서드의 접근제어자가 public이어야 하므로 의도하지 않은 변경 가능성이 존재

        필드 주입 :

            순수한 자바로써 단위 테스트가 불가

            DI을 처리해주는 프레임워크 없이는 인스턴스를 넣어줄 수 없음

    - 의존관계 우선 순위

~~~java
@Component
  public class FixDiscountPolicy implements DiscountPolicy {}

@Component
public class RateDiscountPolicy implements DiscountPolicy {}
    
@Autowired
private DiscountPolicy discountPolicy;


NoUniqueBeanDefinitionException: No qualifying bean of type
'hello.core.discount.DiscountPolicy' available: expected single matching bean
but found 2: fixDiscountPolicy,rateDiscountPolicy
~~~

        같은 타입의 구현체 여러개가 스프링 빈에 등록된 경우 해당 타입으로 의존관계 주입 시 
        NoUniqueBeanDefinitionException 오류가 발생함

        의존관계의 우선 순위를 부여하여 해결 가능

## 자동, 수동의 선택 기준

    - 일반적으로 자동으로 빈에 등록하는 방법이 선호되고 있음

        설정 정보를 기반으로 수동 스프링 빈 등록 시 
        @Bean을 통해 객체를 생성하고 주입할 대상을 일일이 적어줘야하는 번거로움 존재

        관리해야할 빈이 많으면 많아질수록 설정 정보를 관리하는 것 자체가 부담

    - 수동 빈 등록을 사용하는 경우

        기술적인 문제나 공통 관심사(AOP)를 처리할 때 주로 사용
        (데이터베이스 연결, 공통 로그 처리 처럼 업무 로직을 지원하기 위한 하부 기술이나 공통 기술)

            컨트롤러, 서비스, 리포지토리 처럼 어느정도 유사한 패턴이 존재하는 경우 자동 빈 등록을 적극 사용
            기술 지원 로직은 업무 로직과 비교해서 그 수가 매우 적고, 보통 애플리케이션 전반에 걸쳐서 광범위하게 영향을 미침 
            기술 지원 로직은 적용이 잘 되고 있는지 아닌지 조차 파악하기 어려운 경우가 많음
            따라서, 가급적 수동 빈 등록을 사용해서 명확하게 들어내는 것이 좋음

        다형성을 적극 활용할 때 수동 빈 등록에 대한 사용 고려

            의존관계 주입 시 List, Map으로 구현체들을 전부 자동 주입받는 경우 어떤 빈들이 주입될지 파악하기 쉽지 않음
            이와 같이 다형성을 통한 구현체 주입으로 인해 컴파일 시점에 분석하기가 용이하지 않은 경우
            별도의 설정 정보를 만들어 명확하게 드러내는 것이 좋음
            
            설정 정보를 두어 수동으로 빈을 등록하고 싶지 않은 경우
            적어도 주입되는 구현체들의 클래스를 특정 패키지에 같이 묶어두는 것이 좋음
            
    

    
    
