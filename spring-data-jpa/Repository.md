---
sort: 1
---

# Repository

---

## JpaRepository

```java

import org.springframework.data.jpa.repository.JpaRepository;

public interface MemberRepository extends JpaRepository<Member, Long> {}

```

```java

public interface JpaRepository<T, ID extends Serializable> extends PagingAndSortingRepository<T, ID> {
    //...
}

```

    - JpaRepository를 상속받으면 spring data jpa가 Proxy 기법을 이용하여 자동으로 구현체를 만들어줌

    - 자동으로 만들어준 구현체를 통해 대부분의 공통적으로 쓰일만한 메서드를 제공함

    - Repository <- CrudRepository <- PagingAndSortingRepository [spring.data.xxx] 
        <- JpaRepository [spring.data.jpa.xxx]

        JpaRepository의 부모 인터페이스들은 JPA 기술에 국한되어있지 않음

## NamedQuery

```java

public List<Member> findByUsernameAndAgeGreaterThan(String username, int age) {
      return em.createQuery("select m from Member m where m.username = :username and m.age > :age")
              .setParameter("username", username)
              .setParameter("age", age)
              .getResultList();
}

```

    spring data jpa를 이용하지 않고 Repository를 구현하는 경우 메서드를 직접 생성할 수 있으나
    쿼리가 문자열로 인식되기 때문에 컴파일 시점에 해당 쿼리를 검증할 수 없어 오류가 사용자에게 까지 전파될 수 있음

```java

public interface MemberRepository extends JpaRepository<Member, Long> {
    
    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
    
}

```

    JpaRepository를 상속받아 spring data jpa를 이용하는 경우 메서드 이름으로 쿼리 생성이 가능함
    즉, 인터페이스에 추상 메서드를 선언하는 것만으로 spring data jpa가 메서드 이름을 분석하여 적절한 쿼리를 생성함

    엔티티의 필드명 변경 시 인터페이스에 정의한 추상 메서드의 이름도 같이 변경되어야 하는 등
    컴파일 시점에 오류를 방지할 수 있으나 조건이 늘어날수록 메서드 이름이 무한정 길어지게 되는 단점이 존재함
    또한, 복잡한 쿼리 생성에 대한 한계가 존재하므로 조건식이 1~2 이내의 간단한 쿼리 생성 시에 사용

[쿼리 메서드 생성 관련](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)  

```java

@Entity
@NamedQuery(
    name="Member.findByUsername",
    query="select m from Member m where m.username = :username")
public class Member {
    // ... 
}

```

```java

public class MemberRepository {
    public List<Member> findByUsername(String username) {
          List<Member> resultList =
              em.createNamedQuery("Member.findByUsername", Member.class)
                  .setParameter("username", username)
                  .getResultList();
    } 
}

```

```java

@Query(name = "Member.findByUsername")
List<Member> findByUsername(@Param("username") String username);


```

    JPA 표준이 제공하는 NamedQuery 기능 이용 시 작성된 쿼리를 컴파일 시점에 검증이 가능함
    단, NamedQuery를 엔티티에 작성하여야하는 단점이 존재함

    순수 JPA Repository에서 em.createNamedQuery()를 통하여 엔티티에 작성된
    NamedQuery를 사용할 수 있으며 spring data jpa를 이용할 결우
    @Query를 통해 NamedQuery를 사용할 수 있음

        spring data jpa는 설정을 변경하지 않는 한 도메인클래스 + . + 메서드 이름으로 된 
        NamedQuery를 찾아서 실행할 수 있으며 만약 실행할 수 있는 NamedQuery가 없다면 
        메서드 이름을 분석하여 쿼리 생성 전략을 사용함
        따라서, @Query를 생략하고 메서드 이름만으로도 NamedQuery를 호출할 수 있음

    JPA 표준이 제공하는 NamedQuery는 실무에서 잘 사용되지 않으며
    spring data jpa가 제공하는 @Query 기능으로 대체하여 사용됨
    
```java

public interface MemberRepository extends JpaRepository<Member, Long> {
    
    @Query("select m from Member m where m.username= :username and m.age = :age")
    List<Member> findUser(@Param("username") String username, @Param("age") int age);
    
}

```
   
    NamedQuery와 다르게 엔티티가 아닌 Repository에 직접 작성 가능하고
    NamedQuery와 마찬가지로 컴파일 시점에 쿼리 검증이 가능함

```java

public interface MemberRepository extends JpaRepository<Member, Long> {
    
    @Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) from Member m join m.team t")
    List<MemberDto> findMemberDto();   
    
}

```

    jpql에 new 명령어를 사용하여 DTO로 직접 조회할 수 있으며
    작성된 인자들로 구성된 DTO의 생성자가 필요함

## parameter binding

```java

@Query("select m from Member m where m.username in :names")
List<Member> findByNames(@Param("names") Collection<String> names);

```

```java

List<Member> findMembers = memberRepository.findByNames(Arrays.asList("AAA", "BBB"));

```

    spring data jpa가 컬렉션 타입을 이용하여 in절에 적용할 파라미터를 바인딩할 수 있도록 지원함

## return Type

    순수 JPA의 경우 조회된 결과가 없거나 단일 건 조회에 복수의 데이터가 조회되면 예외를 발생시키나
    spring data jpa는 내부적으로 try ~ catch ~ 사용하여 예외 처리 후에 반환 타입에 따라 null 또는 빈 컬렉션 등을 반환함

    - 조회된 결과가 없을 때 반환 타입이 컬렉션 타입이면 null이 아닌 빈 컬렉션을 반환함

    - 조회된 결과가 없을 때 컬렉션 타입이 아닌 일반 객체 타입이면 null을 반환함

    - 일반 객체 타입을 반환할 때 조회된 결과가 2건 이상이면 javax.persistence.NonUniqueResultException 예외 발생

[반환 타입](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repository-query-return-types)

## Page

```java

public List<Member> findByPage(int age, int offset, int limit) {
    return em.createQuery("select m from Member m where m.age = :age order by m.username desc")
            .setParameter("age", age)
            .setFirstResult(offset)
            .setMaxResults(limit)
            .getResultList();
}

public long totalCount(int age) {
        return em.createQuery("select count(m) from Member m where m.age = :age",Long.class)
                .setParameter("age",age)
                .getSingleResult();
}

```

    순수 JPA로 페이징 관련 처리하기 위해선 totalCount와 페이징 처리된 데이터를 조회하여야 함
    이 후, 해당 페이지가 몇번째 페이지인지 조회된 데이터는 몇 개인지 첫번째 페이지, 마지막 페이지 여부 등
    페이징 처리를 위해 totalCount를 이용하여 구현이 추가적으로 이루어져야 함

```java

public interface Page<T> extends Slice<T> {
    int getTotalPages(); //전체 페이지 수
    long getTotalElements(); //전체 데이터 수
    <U> Page<U> map(Function<? super T, ? extends U> converter); //변환기
}


```

```java

public interface Slice<T> extends Streamable<T> {
    int getNumber(); // 현재 페이지
    int getSize(); // 페이지 크기
    int getNumberOfElements(); // 현재 페이지에 나올 데이터 수
    List<T> getContent(); // 조회된 데이터
    boolean hasContent(); // 조회된 데이터 존재 여부
    Sort getSort(); // 정렬 정보
    boolean isFirst(); // 현재 페이지가 첫 페이지 인지 여부
    boolean isLast(); // 현재 페이지가 마지막 페이지 인지 여부
    boolean hasNext(); // 다음 페이지 여부
    boolean hasPrevious(); // 이전 페이지 여부
    Pageable getPageable(); // 페이지 요청 정보
    Pageable nextPageable(); // 다음 페이지 객체
    Pageable previousPageable();//이전 페이지 객체
    <U> Slice<U> map(Function<? super T, ? extends U> converter); // 변환기
}

```

```java

public interface MemberRepository extends Repository<Member, Long> {
       
        Page<Member> findByAge(int age, Pageable pageable);
       
       // or
    
        Slice<Member> findByAge(int age, Pageable pageable);
    
}

```

```java

PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));

Page<Member> page = memberRepository.findByAge(10, pageRequest);

List<Member> content = page.getContent(); //조회된 데이터 
assertThat(content.size()).isEqualTo(3); //조회된 데이터 수
assertThat(page.getTotalElements()).isEqualTo(5); //전체 데이터 수 
assertThat(page.getNumber()).isEqualTo(0); //페이지 번호 
assertThat(page.getTotalPages()).isEqualTo(2); //전체 페이지 번호 
assertThat(page.isFirst()).isTrue(); //첫번째 항목인가? 
assertThat(page.hasNext()).isTrue(); //다음 페이지가 있는가?

```

```java

PageRequest pageRequest = PageRequest.of(1, 3, Sort.by(Sort.Direction.DESC, "username"));

Slice<Member> slice = memberRepository.findByAge(age, pageRequest);

assertThat(slice.getContent().size()).isEqualTo(2);
assertThat(slice.getNumber()).isEqualTo(1);
assertThat(slice.isFirst()).isFalse();
assertThat(slice.hasNext()).isFalse();

```

    spring data jpa를 이용하면 totalCount를 이용한 추가적인 페이징 관련 
    구현을 직접하지 않아도 페이징에 필요한 정보들을 이용할 수 있음

    파라미터

        org.spring.framework.data.domain.Sort : 정렬 기능
        org.spring.framework.data.domain.Pageable : 페이징 기능(내부에 Sort 포함)

    반환타입

        org.springframework.data.domain.Page : 추가 count 쿼리 결과를 포함하는 페이징
        org.springframework.data.domain.Slice : 추가 count 쿼리 없이 다음 페이지만 확인 가능(지정한 limit의 +1 만큼 더 조회함)
        List(자바 컬렉션) : 추가 count 쿼리 없이 결과만 반환

    PageRequest 객체를 이용하여 현재 페이지 수, 조회할 데이터 수를 지정 할 수 있으며
    정렬 정보도 PageRequest 객체에 파라미터로 지정할 수 있음

    Page는 페이징 처리를 위한 totalCount 쿼리를 추가적으로 호출하며
    Slice는 페이징 처리를 하지 않기 때문에 limit + 1 만큼의 데이터를 조회하여
    다음 페이지의 여부만 확인이 가능함

        Page의 경우 추가적으로 totalCount 쿼리르 호출하나 마지막 페이지를 호출하게 되는 경우
        spring data jpa가 마지막 페이지임을 인식하여 totalCount 호출 없이 전체 카운트를 계산할 수 있으므로
        totalCount 쿼리를 호출하지 않음

```java

@Query(value = “select m from Member m”,
         countQuery = “select count(m.username) from Member m”)
  Page<Member> findMemberAllCountBy(Pageable pageable);

```

    Page 기능을 이용하게 되면 totalCount를 자동적으로 호출해주지만 복잡한 쿼리의 경우
    여러 조인들을 성능 최적화없이 그대로 이용하여 count 쿼리를 호출함
    따라서, 쿼리의 복잡도에 따라 성능 최적화가 필요한 경우 countQuery를 직접 작성할 수 있음

```java

Page<Member> page = memberRepository.findByAge(age, pageRequest);

Page<MemberDto> dtoPage = 
        page.map(member -> new MemberDto(member.getId(), member.getUsername(), member.getTeam().getName()));

```

    Page의 map()을 이용하여 엔티티를 특정 DTO 클래스로 변환할 수 있음
    
## bulk update

```java

public int bulkAgePlus(int age){
    int resultCount = em.createQuery(
            "update Member m set m.age = m.age + 1 where m.age >= :age")
        .setParameter("age",age)
        .executeUpdate();
    
    return resultCount;
}

```

```java

@Modifying
@Query("update Member m set m.age = m.age + 1 where m.age >= :age")
int bulkAgePlus(@Param("age") int age);

```

    순수 JPA와 스프링 데이터 JPA를 이용하여 벌크성 쿼리를 호출할 수 있음

    스프링 데이터 JPA의 경우 @Modifying가 필요하며 사용하지 않을 경우 QueryExecutionRequestException 발생

        벌크성 수정, 삭제 시에 @Modifying 어노테이션을 사용함

    벌크성 쿼리는 Persistence Context를 무시하고 쿼리를 수행하기 때문에, 
    엔티티의 상태와 실제 데이터베이스의 데이터의 상태가 달라지게 됨
    따라서, 같은 트랜잭션 내에서 벌크성 쿼리 호출 후에 관련 데이터의 조회가 필요하다면
    Persistence Context를 초기화하여야 함

        Persistence Context 초기화 방법
    
            @Modifying(clearAutomatically = ture)를 통하여 벌크성 쿼리 수행 후에는
            Persistnece Context가 자동으로 초기화되도록 할 수 있음

                clearAutomatically 옵션의 default는 false

            벌크성 쿼리 호출 후 em.clear()

                의도적으로 clear()를 호출하여 Persistence Context를 초기화할 수 있음
                단, FlushModeType 옵션 설정이 Auto가 아니라면 em.claer() 이전에 em.flush()를 호출하여야 함

    Persistence Context에 엔티티가 없는 상태에서 벌크성 쿼리를 호출하는 것을 권장하며
    부득이하게 Persistence Context에 엔티티가 존재한다면 벌크 연산 직후 Persistence Context를 초기화할 것

    참고 :

        JPQL은 Persistence Context를 먼저 조회하지 않고 데이터베이스에서 
        데이터를 조회하여 Persistence Context에 저장하려고 시도함
        이 때, 같은 식별자의 데이터가 Persistnece Context에 이미 존재하는 경우에는
        해당 데이터를 버리는 전략을 취함

## EntityGraph
    
```java

//공통 메서드 오버라이드
@Override
@EntityGraph(attributePaths = {"team"}) 
List<Member> findAll();

//JPQL + 엔티티 그래프 
@EntityGraph(attributePaths = {"team"}) 
@Query("select m from Member m") 
List<Member> findMemberEntityGraph();

//메서드명 분석
@EntityGraph(attributePaths = {"team"}) 
List<Member> findByUsername(String username);

```

    실무에서는 연관관계가 맺어진 엔티티를 조회할 때 지연로딩 전략을 취하므로
    주 엔티티 조회 후에 연관관계가 맺어진 엔티티가 필요할 경우 추가적으로
    관련 엔티티에 대한 쿼리가 호출됨, 이로 인하여 N + 1 문제 발생함
    fetch join을 사용하면 이와 같은 문제를 해결할 수 있고 
    fetch join을 간단히 사용할 수 있도록 @EntityGraph를 제공함

```java

@NamedEntityGraph(name = "Member.all", attributeNodes = @NamedAttributeNode("team"))
@Entity
public class Member {
    // ...
}

```

```java

@EntityGraph("Member.all")
@Query("select m from Member m")
List<Member> findMemberEntityGraph();

```

    JPA 표준에서 제공하는 @NamedEntityGraph를 이용하여 연관관계 엔티티 설정 및 이름을 지정할 수 있으며
    하이버네이트에서 제공하는 @EntityGraph 또한 JPA 표준의 @NamedEntityGraph를 지원함
    @NamedEntityGraph는 JPA 표준에서 제공하는 기능이나 잘 사용되지 않음

    복잡한 쿼리의 경우 @Query를 이용하여 직접 fetch join을 작성하고 상대적으로 간단한 경우 @EntityGraph로 해결할 것

## JPA Hint

```java

@QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))
Member findReadOnlyByUsername(String username);

```

    Persistnece Context는 영속화된 엔티티에 대해서 변경 감지를 위해 스냅샷을 생성함
    완전 조회용으로 사용하는 경우 스냅샷 생성 및 변경 감지를 할 필요가 없으므로
    하이버네이트가 JPA Hint를 통해 성능을 최적화할 수 있도록 제공함(JPA 표준 기능이 아님)

    단, 실질적으로 성능 최적화에 대한 효과는 미미하며 조회에 대한 성능이 안나오는 경우에는
    쿼리 개선 및 레디스를 통한 캐시를 우선 고려해야 함
    
    성능 최적화에 신경쓴다고 조회용 메서드에 모두 JPA Hint를 사용하는 것은 좋지 않은 방법임

## JPA Lock

```java

@Lock(LockModeType.PESSIMISTIC_WRITE)
List<Member> findByUsername(String name);

```

    JPA 표준에서 제공하는 기능으로써, 업데이트를 하기 위한 조회임을 알림으로써
    업데이트가 이루어지기 전까지 외부에서 해당 테이블을 조회하지 못하도록 하는 기능을
    데이터베이스 기능(for update)을 JPA를 통해서 할 수 있도록 지원함

## 사용자 정의 리포지토리

```java

public interface MemberRepositoryCustom {
    
    List<Member> findMemberCustom();
    
}
```

```java

@RequiredArgsConstructor
public class MemberRepositoryImpl implements MemberRepositoryCustom {
    
    private final EntityManager em;
    
    @Override
    public List<Member> findMemberCustom() {
        return em.createQuery("select m from Member m")
                .getResultList();
    } 
    
}

```

```java

public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {
    
}

```

```java

List<Member> result = memberRepository.findMemberCustom();

```

    복잡한 쿼리를 해결하기 위해 스프링 JDBC Templete, MyBatis, Querydsl 등 JPA가 아닌 다른 기술을 사용할 필요가 있음

    대안이 되는 다른 기술들을 Custom 인터페이스 생성 및 impl구현체에 구현

        스프링 데이터 JPA가 관리하는 JpaRepository를 상속받은 인터페이스에 Custom 인터페이스를 상속하면
        스프링 데이터 JPA가 자동으로 인식해서 스프링 빈으로 등록함

        Custom 인터페이스 명은 상관 없으나 impl구현체는 [인터페이스 이름 + Impl] 명명규칙을 따라야 함

            Impl이 아닌 다른 명명규칙을 이용하고 싶다면 xml 또는 JavaConfig 설정을 통해 
            바꿀 수 있으나 관례를 따르는게 유지보수 측면에서 좋음

    스프링 데이터 JPA가 관리하는 인터페이스에 다른 기술을 함께 적용하여 JPA 기술만으로 해결할 수 없는
    복잡한 쿼리를 함께 해결할 수 있다는 장점이 존재하나 반드시 사용자 정의 리포지토리를 이용해야하는 것은 아님

        화면단에 특화된 쿼리나 조회성 쿼리, 핵심 비즈니스 쿼리 등 용도에 따라 
        클래스가 분리되어야 복잡성을 낮추고 유지보수를 수월하게 할 수 있음
        용도 또는 특성에 맞게 클래스는 분리되어야 하며 특히 화면단에 특화된 쿼리 같이
        복잡한 쿼리 특성으로 인하여 다른 기술을 함께 적용해야 한다면 사용자 정의 리포지토리가
        아닌 클래스 자체를 분리하면서 해당 클래스에 다른 기술을 구현하면 되는 것임
        
## Auditing

```java

@MappedSuperclass
@Getter
public class JpaBaseEntity {
        
    @Column(updatable = false)
    private LocalDateTime createdDate;
    private LocalDateTime updatedDate;
    
    @PrePersist
    public void prePersist() {
        LocalDateTime now = LocalDateTime.now();
        createdDate = now;
        updatedDate = now;
    }
    
    @PreUpdate
    public void preUpdate() {
        updatedDate = LocalDateTime.now();
    }
    
}

```

    순수 JPA의 이벤트를 이용하여 등록일, 수정일이 자동으로 생성되도록 처리할 수 있음

    @PrePersist, @PreUpdate, @PostPersist, @PostUpdate 이벤트를 엔티티 속성에 적용

```java

public class BaseTimeEntity {
    
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;
    
    @LastModifiedDate
    private LocalDateTime lastModifiedDate;

}

public class BaseEntity extends BaseTimeEntity {
    
    @CreatedBy
    @Column(updatable = false)
    private String createdBy;
    
    @LastModifiedBy
    private String lastModifiedBy;
    
}

```    

```java

@Bean
public AuditorAware<String> auditorProvider() {
    return () -> Optional.of(UUID.randomUUID().toString());
    // 실무에서는 세션 또는 스프링 시큐리티에서 ID 정보를 반환하여야 함
}

```

    스프링 데이터 JPA를 이용하여 등록일, 수정일, 등록자, 수정자가 자동으로 생성되도록 처리할 수 있음

        @EnableJpaAuditing을 스프링 부트 설정 클래스에 적용
        @CreatedDate, @LastModifiedDate, @CreatedBy, @LastModifiedBy를 엔티티 속성에 적용
        @EntityListeners(AuditingEntityListener.class)를 엔티티에 적용

            META-INF/orm.xml에 AuditingEntityListener를 이용하여 모든 엔티티에 공통 적용할 수 있음

    모든 엔티티에 적용될 가능성이 높은 등록일, 수정일 필드를 최상위 클래스에 두고 엔티티 특성에 따라
    사용하지 않을 수도 있는 등록자, 수정자는 클래스를 따로 만들어 상속받도록 하는 것이 좋음

    등록자, 수정자를 처리하기 위해서는 AuditorAware를 스프링 빈에 등록하여야 하며
    실무에서는 해당 빈에서 세션 정보나 스프링 시큐리티 로그인 정보를 통해 ID를 리턴해주어야 함

    데이터 저장 시점에 업데이트 관련 필드에는 데이터를 입력하고 싶지 않다면
    @EnabledJpaAuditing(modifyOnCreate = false)옵션을 사용하면 됨

## 도메인 클래스 컨버터

```java

@RestController
@RequiredArgsConstructor
public class MemberController {
    
    private final MemberRepository memberRepository;
    
    @GetMapping("/members/{id}")
        public String findMember(@PathVariable("id") Member member) {
        return member.getUsername();
    }
    
}

```    

    엔티티를 파라미터로 이용할 시 스프링 데이터 JPA가 컨버터를 이용하여 관련된 엔티티를 주입함
    
    컨버터를 통하여 얻어낸 엔티티는 단순 조회용으로 사용하여야 함
    OSIV가 활성화되어 있지 않는 한 컨트롤러단으로 넘어온 순간
    트랜잭션이 끊기게 되어 준영속 상태로 전환됨

    트랜잭션 관련 문제로 인하여 사용하는 것을 권장하지 않음

## 페이징, 정렬

```java

@GetMapping("/members")
public Page<Member> list(Pageable pageable) {
    return memberRepository.findAll(pageable);
}

```

```java

@RequestMapping(value = "/members_page", method = RequestMethod.GET)
public String list(@PageableDefault(size = 12, sort = “username”, 
        direction = Sort.Direction.DESC) Pageable pageable) {
    
    // ...
        
}

```

    Pageable을 파라미터로 이용할 시 스프링 데이터 JPA가 쿼리 파라미터를 분석하여
    내부적으로 PageRequest를 생성하고 페이징 처리할 수 있도록 함

    @PageableDefault를 통하여 매핑되는 메서드마다 개별적으로 페이징 설정 가능

    페이징 정보가 둘 이상일 경우 @Qualifier 이용

## Persistable

```java

public interface Persistable<ID> {
    
    ID getId();
    boolean isNew();
    
}

```

```java

@EntityListeners(AuditingEntityListener.class)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Item implements Persistable<String> {
    
    @Id
    private String id;
    
    @CreatedDate
    private LocalDateTime createdDate;
    
    public Item(String id) {
        this.id = id;
    }
    
    @Override
    public String getId() {
        return id; 
    }
    
    @Override
    public boolean isNew() {
        return createdDate == null;
    }

}

```

    설계 상 엔티티의 PK가 Long 타입이 아닌 String 타입이거나 @GenerateValue를 이용하지 않고
    식별자를 직접 생성 후 save()를 호출하는 경우 스프링 데이터 JPA가 이미 존재하는 엔티티로 인지하여
    merge()를 호출함

    merge()가 호출되면 해당 식별자로 데이터베이스를 조회하게 되고 데이터가 존재하지 않을 시 
    새로운 엔티티라는 것을 인지하게 되므로 매우 비효율적임

    따라서, JpaRepository 구현체인 SimpleJpaRepository의 save 메서드 내에
    새로운 엔티티인지 판단하는 isNew 메서드를 Persistable을 통해 직접 구현할 수 있도록 제공함

    엔티티가 Persistable을 구현하도록 하여 isNew()를 재정의하면 되는데
    실무에서는 createdDate가 null인지 아닌지에 따라 새로운 엔티티 여부를 판단함

## Projections

```java

<T> List<T> findProjectionsByUsername(String username, Class<T> type);

```

```java

public interface UsernameOnly {
   
    String getUsername();
    
}

public interface UsernameOnly {
    
    @Value("#{target.username + ' ' + target.age + ' ' + target.team.name}")
    String getUsername();
    
}

public class UsernameOnlyDto {
    
    private final String username;
    
    public UsernameOnlyDto(String username) {
        this.username = username;
    }
    
    public String getUsername() {
        return username;
    } 
    
}

```

    전체 엔티티 조회가 아닌 DTO로 조회하거나 일부 필드만 조회하고 싶을 때 Projections를 이용함

    - 인터페이스 기반 Projections

        인터페이스 안에 엔티티의 필드명과 일치하는 get메서드가 있어야 함
        인터페이스 기반 Projections를 작성해두면 스프링 데이터 JPA가 
        구현체를 자동 생성함

        - Close Projections : 프로퍼티 형식의 인터페이스 제공(get메서드), 최적화된 쿼리 호출
        - Open Projections : @Value를 이용하여 SpEL문법 사용 가능, 엔티티에 작성된 필드 전부 쿼리로 호출

    - 클래스 기반 Projections

        DTO 클래스에 작성된 속성이 엔티티의 필드명과 일치한다면 DTO타입으로 조회할 수 있음
        생성자 파라미터 이름으로 매칭하기 때문에 생성자의 파라미터명이 엔티티 필드명과 정확히 일치하여야 함

    인터페이스, 클래스 기반과 관계없이 연관관계가 맺어진 엔티티를 조회하는 경우 해당 엔티티에 작성된 필드 전부 쿼리로 호출

    연관관계 엔티티 없이 주 엔티티 관련 필드만 조회하는 경우 사용 