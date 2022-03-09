---
sort: 1
---

# Querydsl

---

    - Q타입
        - Querydsl 사용을 위한 셋팅 (jpql 로깅 시 p6spy null 현상)
        - Q타입 사용법 (JPAQeuryFactory -> 필드 레벨에 작성, Q타입 static import)
        - Querydsl 사용 시 변환된 jpql확인을 위한 설정

    - 검색 조건(where절  and 체이닝)
    - 결과 조회(fetchResults(), fetchCount() deprecated)
    - 정렬
    - 페이징 쿼리
    - 집합(tuple - 직접 지정한 것이 여러개면 tuple, 직접 지정한 것이 1개면 해당 타입으로)
    - Group by, having

    - join(inner, left, right, cross)
        - 기존에는 연관관계가 없는 필드는 cross join을 통하여 데이터를 조회하였으나
        - 하이버네이트가 on절을 통해서 연관관계가 없는 필드도 외부 조인을 할 수 있도록 개선함

    - on절
        - on절을 통한 외부 조인 조건 검색
            - id를 통한 조인은 따로 작성하지 않아도 알아서 작성해줌(join관련 메서드에 연관관계가 있는 필드를 정확히 매칭한 경우)
            - Inner join의 경우 on절을 통해 조건을 거나 where절에서 조건을 거나 차이가 없음
            - left join과 같이 null 데이터를 포함해야하는 경우 on절에서 조건을 걸어야 함
        - on절을 통한 연관관계 없는 엔티티 조인
            - Inner join의 경우도 on절에 조건을 걸어야 함

    - Fetch 조인 사용법
    - 서브쿼리 사용법
    - case문 사용법
    - 상수, 문자 더하기 사용법

    - 프로젝션 기본(엔티티, 기본타입, tuple)
    - 프로젝션 dto(setter, field, constructor)
        - 엔티티 필드명과 일치하지 않는 dto 변수명 처리
            - Constructor 방식은 타입만 일치하면 되기때문에 필드명과 변수명이 달라도 가능
    - @QueryProjection
        - 장점: 생성자 방식과 비교하면 생성자 방식은 생성자에 해당하지 않는 파라미터 추가 시 컴파일 단계에서 오류를 잡아내지 못함
        - @QueryProjection은 Q타입 객체를 만들어 인스턴스화 하므로 생성자에 맞지 않는 파라미터 전달 시 컴파일 오류 발생
        - 단점: Q타입 객체를 생성해야하는 비용 발생, DTO가 queryDsl을 의존하게 됨

    - 동적 쿼리
        - BooleanBuilder -> 쿼리를 직관적으로 파악하기 힘듦
        - 다중 where -> 쿼리 직관적으로 파악 용이, 동적 쿼리에 대한 조합이 가능, 재사용성 높음

    - 수정, 삭제 벌크 연산

    - Querydsl, spring data jpa 구성 방법
    - 특정 기능에 특화되어 있는 경우 클래스로 repository 생성하여 만들어도 무방하다 (이전 강의 내용 정리한 부분)

    - Querydsl paging 사용
        - Query results, query count deprecated -> 필요한 경우 countquery 분리
        - count(*)를 사용하고 싶은 경우 Wildcard.count()를 이용
        - 결과값이 숫자 하나 이므로 fetchOne() 이용

    - QuerydslPredicateExecutor
        - 장점 : springdata가 자동으로 만들어주는 메서드에 인자로 predicate를 넣어줄 수 있음
        - 단점:
            - 조인 불가(묵시적 조인은 가능하나 외부 조인 불가함
            - Querydsl 의존성 증가(다른 계층 클래스들이 querydsl 기술을 이용하기 위해 predicate 인자를 만들어야 함)

    - Querydsl web
        - 장점 : 컨트롤러 호출 시 넘어온 인자를 querydsl 조건 검색 형태로 바꾸어줌
        - 단점 :
            - 컨트롤러가 querydsl에 의존
            - 조건에 대한 커스텀 기능이 복잡, 명시적이지 않음

    - querydslRepositorySupport
        - 장점:
            - 스프링 데이터가 제공하는 페이징을 querydsl로 편리하게 변환 가능
            - Entity manager 제공
        - 단점 :
            - Querydsl 3.x 버전 대상
                - from절 부터 시작했기 때문에 querydslRepositorySupport 또한 from 절부터 시작됨
                - JPAQueryFactory 대응 안되어있음
            - Sort 기능 정상 동작하지 않음

