---
sort: 1
---

# PersistenceContext

---

## 초기 설정

    - JPA Hibernate 메이븐 등록

```xml

    <dependencies>

        <!-- JPA 하이버네이트 -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>5.6.3.Final</version>
        </dependency>

        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.3.1</version>
        </dependency>

    </dependencies>

```

        개발환경에 맞추어 연결할 데이터베이스 및 하이버네이트 dependency 등록함


    - persistence.xml 등록

```xml
    
    <?xml version="1.0" encoding="UTF-8"?>
    <persistence version="2.2"
                 xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
        <persistence-unit name="jpa_study">
            <properties>
                <!-- 필수 속성 -->
                <property name="javax.persistence.jdbc.driver" value="org.postgresql.Driver"/>
                <property name="javax.persistence.jdbc.user" value="jpa_user"/>
                <property name="javax.persistence.jdbc.password" value="jpa_user"/>
                <property name="javax.persistence.jdbc.url" value="jdbc:postgresql://localhost:5432/jpa_study"/>
                <property name="hibernate.dialect" value="org.hibernate.dialect.PostgreSQL10Dialect"/>
                <!-- 옵션 -->
                <property name="hibernate.show_sql" value="true"/>
                <property name="hibernate.format_sql" value="true"/>
                <property name="hibernate.use_sql_comments" value="true"/>
                <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
            </properties>
        </persistence-unit>
    </persistence>

```

        classPath 하위의 META-INF 안에 persistence.xml 생성

        javax.persistence로 시작하는 속성은 JPA 표준 속성으로써
        현재 구현체인 Hibernate에서 다른 구현체로 변경되어도 정상 동작,
        hibernate로 시작하는 속성은 Hibernate 전용 속성으로써
        다른 구현체로 변경되면 정상 동작하지 않음

        * hibrernate.dialect 
            
            Hibernate에서 제공하는 데이터베이스 방언 지원 기능

            SQL 표준을 지키지 않는 특정 데이터베이스만의 고유 기능 존재,
            JPA는 특정 데이터베이스에 종속되지 않으므로 각각의 데이터베이스가 
            제공하는 상이한 SQL 문법과 함수를 지원함
            
                가변 문자: MySQL -> VARCHAR, Oracle -> VARCHAR2
                문자열 자르기 : SQL 표준 -> SUBSTRING(), Oracle -> SUBSTR()
                                    ...

## 영속성 컨텍스트

    - PersistenceContext : "엔티티를 영구 저장하는 환경"

        EntityManger를 통하여 PeristenceContext에 접근

```java

    EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpa_study");
    EntityManager em = emf.createEntityManager();
    
    EntityTransaction ts = em.getTransaction();
    ts.begin();
    
    try {
        
        // ...
        
        ts.commit();
    } catch (Exception e) {
        ts.rollback();
    } finally {
        em.close();
        emf.close();
    }

```

        사용자 요청 시에 트랜잭션 단위로 EntityManger가 생성되며
        EntityManger를 통하여 PersistenceConext에 접근 및 
        커넥션풀을 사용하여 DB에 데이터를 생성, 조회, 수정, 삭제함
    
        일반적인 환경에서는 EntityManger와 PersistenceContext가 1:1 관계이나 
        스프링 프레임워크 같은 컨테이너 환경에서는 EntityManger와 PersistenceContext가 N:1 관계가 됨

### 엔티티 생명주기

    - 비영속(new/transient)

        PersistenceContext와 전혀 관계가 없는 새로운 상태(new) 혹은 관리 대상 제외(transient)

```java
    Member member = new Member();
    member.setId(1L);
    member.setUsername("member1");
```

    - 영속(managed)
 
        PersistenceContext에 관리되는 상태

```java
    em.persist(member);
```

    - 준영속(detached)

        PersistenceContext 관리 대상이였다가 분리된 상태

```java

    // 특정 엔티티만 준영속 상태로 전환
    em.detach(member);

    // PersistContext 초기화
    em.clear();
    
    // PersistContext 종료
    em.close();

```

    - 삭제(removed)

        persistenceContext에 관리되는 엔티티 삭제 및 데이터베이스 데이터 삭제

```java
    em.remove(member);
```

### 1차 캐시

    - PersistenceContext는 1차 캐시를 지원함

```java

    Member member = new Member();
    member.setId(1L);
    member.setUsername("member1");
            
    //1차 캐시에 저장됨
    em.persist(member);
            
    //1차 캐시에서 조회
    Member findMember = em.find(Member.class, 1L);

```

        persist() 시에 데이터베이스에 해당 엔티티를 데이터베이스에 저장시키는 것이 아니고
        1차 캐시에 해당 객체의 @Id를 키 값으로 해당 객체를 저장함

        데이터베이스에서 데이터 조회 시에도 곧바로 데이터베이스에서 데이터를 조회해오는 것이 아닌
        1차 캐시에서 엔티티가 존재하는지 여부 판단 후에 존재하지 않으면 데이터베이스에서 조회함
        
        persist시에 Id가 1인 엔티티가 1차 캐시에 저장되었으므로
        find를 통한 조회 시에 데이터베이스에서 조회해오는 것이 아닌
        1차 캐시에서 해당 엔티티를 찾아 반환함

        만약 find시에 1차 캐시에 존재하지 않는 엔티티라면 데이터베이스를 조회하고
        얻어온 엔티티를 1차 캐시에 저장 후에 반환함

### 엔티티 동일성 보장

    - PersistencdContext는 1차 캐시를 지원하기 때문에 엔티티에 대한 동일성을 보장함

```java

    Member a = em.find(Member.class, 1L);
    Member b = em.find(Member.class, 1L);
            
    System.out.println(a == b); //동일성 비교 true

```

        find를 통한 조회 시에 1차 캐시에서 같은 엔티티가 반환

### 쓰기 지연

    - persist(), setXxx(), remove() 등 데이터 조작 시에 쓰기 지연 SQL 저장소에 저장됨

        데이터를 추가하는 persist(), 데이터를 수정하는 setXxx(), 데이터를 삭제하는 remove()는
        호출 시에 바로 SQL문이 데이터베이스에 수행되는 것이 아닌 1차 캐시에 엔티티 저장 및
        쓰기 지연 SQL 저장소에 데이터베이스에 수행되어야할 SQL문을 저장함

        이 후, transaction.commit()이 호출되어야 비로소 쓰기 지연 SQL 저장소에 저장된
        SQL문이 flush(), commit()됨

### 변경 감지

    - setXxx()와 같이 데이터 수정 시에는 변경을 감지하기 위해 Dirty Checking함

        Dirty Checking 시에 변경이 되었는지 비교하기 위한 초기 엔티티 상태를
        1차 캐시에 스냅샷이라는 곳에 저장함

        이 후, setXxx()가 호출되면 Dirty Checking을 통하여 스냅샷과 엔티티를 비교하여
        수정된 엔티티를 1차 캐시에 저장 및 update SQL문을 쓰기 지연 SQL 저장소에 저장함

## flush

    - 쓰기 지연 SQL 저장소의 SQL문을 데이터베이스에 전송

        SQL문이 실행될 뿐, transaction이 commit되는 것은 아님
        
        transaction.commit() 시 flush()가 자동 호출됨

        flush 호출 : 
            
            수동 호출 -> em.flush() 
            자동 호출 -> transaction.commit(), JPQL Query 실행

    - JPQL Query 실행

```java

    em.persist(memberA);
    em.persist(memberB);
    em.persist(memberC);
    
    List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();

```

        쓰기 지연으로 인하여 persist()시에 1차 캐시에 저장되고 쓰기 지연 SQL 저장소에 SQL문 저장되나
        JPQL Query가 실행된 순간 flush()가 자동 호출되어 SQL문이 데이터베이스에 전송됨

        데이터베이스가 커밋된 상태는 아니므로 실제 데이터베이스에 데이터가 반영된 상태는 아니나
        같은 트랜잭션 안에 존재하기 때문에 JPQL Query를 통해 얻어온 데이터에는
        memberA, memberB, memberC의 엔티티가 담겨져 있음