---
sort: 2
---

# 빈 생명주기 콜백

---

## 콜백 호출 방법
    
라이프 사이클

    - 스프링 빈의 이벤트 라이프 사이클

        스프링 컨테이너 생성 -> 스프링 빈 생성 -> 의존관계 주입 
            -> 초기화 콜백 -> 사용 -> 소멸전 콜백 -> 스프링 종료

        초기화 콜백 : 빈이 생성되고, 빈의 의존관 주입이 완료된 후 호출
        소멸전 콜백 : 빈이 소멸되기 직전에 호출

인터페이스 InitializingBean, DisposableBean

    - 스프링 전용 인터페이스

        스프링 전용 인터페이스로써 외부 라이브러리에 적용할 수 없음
        
    - 콜백 메서드
        
        InitializingBean -> afterPropertiesSet()
        DisposableBean -> destroy()

        초기화와 소멸 메서드의 이름을 변경할 수 없음
    

빈 초기화, 소멸 메서드 지정

    -

@ PostConstruct, @ PreDestroy

    -





