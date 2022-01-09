---
sort: 6
---

# [JPA]

    JPA, Java Persistence API : 자바 진영의 ORM 기술 표준

        - 인터페이스 모음
        - JPA2.1 표준 명세를 구현한 구현체 : Hibernate, EclipseLink, DataNucleus

        - JPA1.0(JSR 220) 2006년 : 초기 버전, 복합 키와 연관관계 기능이 부족
        - JPA2.0(JSR 317) 2009년 : 대부분의 ORM 기능을 포함, JPA Critetia 추가
        - JPA2.1(JSR 338) 2013년 : 스토어드 프로시저 접근, 컨버터, 엔티티 그래프 기능 추가

    ORM, Object-relational mapping : 객체 관계 매핑

        - 객체는 객체대로, 관계형 데이터베이스는 관계형 데이터베이스대로 설계
        - ORM 프레임워크가 중간에서 객체와 관계형 데이터베이스를 매핑
        - 대중적인 언어에는 대부분 ORM 기술이 존재함

    객체와 관계형 데이터베이스 간의 패러다임 불일치로 인하여 기존에는
    객체를 관계형 데이터베이스의 구조에 맞추어 설계하였음

    SQL 중심적인 개발, 객체지향적이지 못한 설계가 문제로써
    ORM 기술을 적용한 JPA를 이용하여 패러다임 불일치 문제를 해결


{% include list.liquid all=true %}
