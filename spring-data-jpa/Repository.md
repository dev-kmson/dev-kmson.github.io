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
    