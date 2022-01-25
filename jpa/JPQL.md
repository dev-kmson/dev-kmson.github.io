---
sort: 10
---

# JPQL

---

    - JPQL(Java Persistence Query Language)

        테이블 대상으로 쿼리하는 SQL과 다르게 엔티티 객체를 대상으로 쿼리함

```java

    // 반환 타입이 명확할 때
    TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m", Member.class);

    // 반환 타입이 명확하지 않을 때
    // Object[] 또는 대응되는 DTO 생성하여 받음
    Query query = em.createQuery("SELECT m.username, m.age from Member m");
    
    // 결과가 하나 이상일 때, 리스트 반환
    // 결과가 없으면 빈 리스트 반환
    query.getResultList();
    
    // 결과가 정확히 하나, 단일 객체 반환
    // 결과가 없으면: javax.persistence.NoResultException
    // 둘 이상이면: javax.persistence.NonUniqueResultException
    query.getSingleResult();

    // 파라미터 바인딩
    TypeQuery<Member> query = em.createQuery(SELECT m FROM Member m where m.username=:username, Member.class); 
    query.setParameter("username", usernameParam);

    // 페이징 쿼리
    String jpql = "select m from Member m order by m.name desc"; 
    List<Member> resultList = em.createQuery(jpql, Member.class)
        .setFirstResult(10)
        .setMaxResults(20)
        .getResultList();

    
    // --------------------------------------------------------------

            // 조인 대상 테이블이 m.team t인 경우 연관관계 PK, FK 매핑 자동 처리됨
            // 조인 대상 테이블이 Team t인 경우 연관관계 PK, FK 매핑 처리되지 않음
            
    // 내부 조인
    SELECT m FROM Member m [INNER] JOIN m.team t ON t.name = 'A'
            
    // 외부 조인
    SELECT m FROM Member m LEFT [OUTER] JOIN m.team t ON t.name = 'A'
            
    // 세타 조인 
    SELECT count(m) FROM Member m, Team t WHERE m.username = t.name
    
    // 연관관계 없는 엔티티 외부 조인
    SELECT m, t FROM Member m LEFT JOIN Team t ON m.username = t.name

    // --------------------------------------------------------------


            // 서브쿼리(JPQL을 통해서는 FROM절에서 서브쿼리 사용 불가 -> 조인으로 해결해야 함)
        // [NOT] EXISTS (subquery): 서브쿼리에 결과가 존재하면 참 
        // {ALL | ANY | SOME} (subquery)
            // ALL 모두 만족하면 참
            // ANY, SOME: 같은 의미, 조건을 하나라도 만족하면 참
        // [NOT] IN (subquery): 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참
    SELECT m FROM Member m WHERE m.age > (SELECT avg(m2.age) FROM Member m2)

    // CASE식
    SELECT
        case when m.age <= 10 then '학생요금'
             when m.age >= 60 then '경로요금'
             else '일반요금'
        end
    FROM Member m

    // COALESCE : null이면 두번째 인자로 대체
    SELECT coalesce(m.username, '이름 없는 회원') FROM Member m

    // NULLIF : 두번째 인자와 동일하면 NULL 반환
    SELECT NULLIF(m.username, '관리자') FROM Member m

    // 데이터베이스 함수 이용
    // 참고 : 데이터베이스 함수를 이용하기 위해선 사용하는 DB의 방언 상속받아 함수를 별도로 등록하여야함
    SELECT function('group_concat', i.name) FROM Item i

    // --------------------------------------------------------------

            // JPQL을 통하여 연관필드를 조회하면 묵시적 내부 조인이 발생
            // 묵시적으로 내부 조인이 발생하게되면 파악이 용이하지 않으므로
            // 명시적 조인을 사용해야 함

    // 단일 값 연관 필드 조회(묵시적 내부 조인 발생)
    SELECT o.member FROM Order o

    // 컬렉션 연관 필드 조회(묵시적 내부 조인 발생)
    SELECT t.members FROM Team

    // --------------------------------------------------------------

    // --------------------------------------------------------------

            // 패치 조인을 이용하면 연관된 엔티티 조회 시도 시
            // 지연 로딩을 통하여 추가적으로 발생하는 쿼리 대신
            // 연관된 엔티티를 한번에 조회할 수 있음
            
            // 패치 조인은 별칭을 주지 않는 것이 원칙
            // JPA는 패치 조인 이용 시 연관된 엔티티를 
            // 전부 가져오는 것을 원칙으로 설계가 되어있음
            // 별칭을 통하여 조건문을 통해 연관된 엔티티의 일부만
            // 가져온다는 것은 의도한 설계가 아닐 뿐더러 엔티티 조작 시
            // 문제가 발생할 여지가 있음
            
            // 일대다 fetch join 시에는 페이징을 이용할 수 없음
            // JPA는 연관된 엔티티를 전부 가져오는 것을 원칙으로
            // 설계가 되어 있으므로 페이징 처리를 한다는 것 자체가
            // 원칙을 위배함을 의미함
            // 따라서, JPA 구현체인 하이버네이트는 일대다 fetch join 시
            // 페이징 처리가 되어있으면 데이터베이스에서 페이징 처리없이
            // 연관된 모든 엔티티를 가져오고 메모리 상에서 페이징을 처리함
            // 즉, 관련 테이블의 데이터 건수가 방대할수록 성능 이슈 및
            // 의도치 않은 성능 저하가 일어남
            
    // fetch join 
    // 패치 조인 사용 시 마치 즉시로딩처럼 연관된 엔티티를 함께 조회함
    // Member 뿐만 아니라 Team에 대한 정보도 동시에 가지고 옴
    // 따라서, Team에 대한 정보가 필요할 시 추가적인 쿼리 호출 없이
    // fetch join을 통하여 얻어온 Team의 정보를 바로 이용할 수 있음
    SELECT m FROM Member m join fetch m.team
    
    // 일대다 fetch join
    // t를 통하여 members를 얻어낼 시 같은 팀인 경우에는
    // members에 대한 정보가 모두 동일할 수 밖에 없음
    SELECT t FROM Team t join fetch t.members
            
    // 일대다 fetch join -> DISTINCT
    // DISTINCT를 통하여 같은 같은 식별자를 가진 t를 제거할 수 있음 
    // 실제 데이터베이스 상에서는 같은 팀을 가진 멤버 식별자가 다른 데이터이므로
    // DISTINCT하여도 변화가 없지만, JPQL에서 DISTINCT를 하면 JPA가
    // 같은 식별자를 가진 엔티티를 중복으로 간주하여 제거함
    SELECT DISTINCT t FROM Team t join fetch t.members
    
    // --------------------------------------------------------------
            
    // 상속 관계에 있는 엔티티 조회
    SELECT i FROM Item i WHERE type(i) IN (Book, Movie)
    SELECT i FROM Item i WHERE treat(i as Book).auther = ‘kim’
    
    // Named Query
    // 어노테이션, XML을 이용하여 정의
    // 애플리케이션 로딩 시점에 초기화 후 재사용 함
    // 애플리케이션 로딩 시점에 쿼리를 검증할 수 있음
    @Entity
    @NamedQuery(
            name = "Member.findByUsername",
            query="select m from Member m where m.username = :username")
    public class Member {
        ...
    }
        
    List<Member> resultList =
        em.createNamedQuery("Member.findByUsername", Member.class)
            .setParameter("username", "회원1")
            .getResultList();

             
    // 벌크 연산
    // 단일건이 아닌 UPDATE, DELETE 쿼리를 지원
    // PersistenceContext를 통하지않고 데이터베이스에 직접 쿼리 호출하므로 
    // 벌크 연산 이후에는 PersistenceContext 초기화 필요(em.clear())     
    String qlString = "update Product p " +
            "set p.price = p.price * 1.1 " +
            "where p.stockAmount < :stockAmount";
    
    int resultCount = em.createQuery(qlString)
            .setParameter("stockAmount", 10)
            .executeUpdate();
```