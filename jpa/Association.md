---
sort: 3
---

# Association

---

    - 방향(Direction) : 단방향, 양방향

    - 다중성(Muitiplicity) : 다대일(N:1), 일대다(1:N), 일대일(1:1), 다대다(N:M)

    - 연관관계의 주인(Owner) : 객체 양방향 연관관계에서의 주인

## 모델링

### 데이터 지향 모델링

```java

    Team team = new Team();
    team.setName("TeamA");
    em.persist(team);
    
    Member member = new Member();
    member.setName("Member1");
    member.setTeamId(team.getId());
    em.persist(member);
    
```

    Team Entity를 Persist()하면 1차 캐시에 Id값과 Entity가 저장됨
    저장된 Id값을 이용하여 Member 엔티티에 외래키로써 저장하였음

    객체지향적이라면 객체간의 참조를 통하여 연관된 객체를 찾아야하지만
    데이터 지향 모델링으로 설계한 순간 마치 관계형 데이터베이스의 외래키를
    통해 연관된 테이블을 찾는 듯이 동작하고 있음을 알 수 있음

### 객체 지향 모델링

```java
    
    Team team = new Team();
    team.setName("TeamA");
    em.persist(team);
    
    Member member = new Member();
    member.setName("member1");
    member.setTeam(team);
    em.persist(member);

```

    JPA를 이용하면 관계형 데이터베이스 모델을 따르지 않고
    객체 지향적으로 설계하여도 ORM 기술이 적용되어 객체 지향과
    데이터 지향의 간극을 메꿔줌
    
## 다중성

### 다대일(N:1)

#### 단방향

```java
    
    @Entity
    @Table(name = "ORDERS")
    public class Order {
    
        @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
        @Column(name = "ORDER_ID")
        private Long id;
        
        @ManyToOne
        @JoinColumn(name = "MEMBER_ID")
        private Member member;
        
    }

```

    가장 많이 사용되는 연관관계로써 다대일 관계에서는 항상 '다'가 연관관계의 주인이 되어야 함
    실제로 관계형 데이터베이스의 테이블 구조 상으로도 '다'쪽의 테이블이 외래키를 들고 있어야 함
    '일'쪽이 외래키를 가지게 되는 순간 '다'에 해당하는 데이터를 '일'의 PK 중복을 통해
    쌓여야 하는 상황이 되므로 설계 구조 상 맞지 않음

    따라서, @ManyToOne을 이용하고 name 속성을 통하여 외래키를 지정함으로써 연관관계의 주인으로 설정함

    일반적으로 특수한 상황이 아니고서는 다대일의 연관관계를 사용하며 초기 설계 시에
    단방향으로 설계 후 '일'쪽에서 '다'의 데이터 조회가 필요하다 판단 시 양방향으로
    전환하는 것이 좋음

#### 양방향

```java

    @Entity
    @Table(name = "ORDERS")
    public class Order {
    
        @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
        @Column(name = "ORDER_ID")
        private Long id;
        
        @ManyToOne
        @JoinColumn(name = "MEMBER_ID")
        private Member member;
        
        // ...
        
    }

```

```java

    @Entity
    public class Member {
    
        @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
        @Column(name = "MEMBER_ID")
        private Long id;
    
        @OneToMany(mappedBy = "member")
        private List<Order> orders = new ArrayList<>();
    
        // ...
        
    }

```

    양방향 지정 시에는 @OneToMany를 이용하고 mappedBy 속성을 통하여 외래키로 지정된 객체를 기입함 
    
    mappedBy 속성을 사용한다는 것은 곧 연관관계의 주인이 아님을 의미하며
    엔티티의 등록, 수정, 삭제가 불가하며 오직 읽기만 가능하게 됨
    즉, 그저 반대방향으로의 조회 탐색 기능만 추가된 것이므로 
    단방향으로 설계 후에 필요 시에 양방향으로 전환하여도 무방함
    
```java

    Member member = new Member();
    member.setName("memberA");
    em.persist(member);
    
    Order order = new Order();
    order.setName("order1");
    order.setMember(member);
    em.persist(order);

    Member findMember = em.find(Member.class, member.getId());
    List<Order> orders = findMember.getOrders();
    for (Order o : orders) {
        System.out.println("o.getName() = " + o.getName());    
    }
    
```

    양방향 사용 시에는 주의할 점이 있는데 이는 양쪽 모두에 값 세팅이 필요하다는 점임
    위와 같이 order 엔티티에 member 엔티티를 참조할 수 있도록 추가한 경우
    JPA가 member 엔티티의 Id를 외래키로 데이터베이스에 등록시키지만
    이 후에, 조회를 통하여 1차 캐시에서 얻어온 member 엔티티에는 
    양방향으로 설정되어 있더라도 order 엔티티에 대한 정보가 담겨있지 않음
    
    만약, em.flush(), em.clear()를 한다면 데이터베이스에 다시 조회해오기 때문에
    JPA가 양방향 연관관계를 인식하여 order 엔티티에 대한 정보를 담아주지만
    그렇지 않은 경우에는 1차 캐시에서 바로 조회해오기 때문에 order 엔티티에 대한
    정보를 가져올 수 없음

    따라서, 양쪽 모두에 값 세팅이 필요하며 위의 이유가 아니더라도
    객체지향적으로 봤을 때 양쪽 모두에 값을 세팅하여 데이터를 일치시키는게 타당함

    양쪽 모두 값을 세팅하기 위해 order.setMember(member), Member.getOrders.add(order)와
    같이 반복적인 작업이 필요하기 때문에 어느 하나를 작성하지 않아 누락시키는 실수를 미연에 방지하기 위하여
    연관관계 편의 메소드를 만들어 두 엔티티 중 한쪽에서 두 값을 세팅해주는 작업을 하도록 하는 것이 좋음

### 일대다(1:N)

#### 단방향
    
    일대다 단방향은 다대일 단방향과 정반대로 '일'이 연관관계의 주인임을 의미함
    그러나 관계형 데이터베이스의 테이블 구조 상 항상 '다'쪽에 외래 키가 존재하므로
    연관관계의 주인인 '일'이 반대편 테이블('다')의 외래 키를 관리하는 특이한 구조가 됨

    다대일 연관관계에서는 @JoinColumn을 사용하지 않아도 반대편 테이블의 PK와 컬럼을 연결하지만
    일대다 연관관계에서는 @JoinColumn을 사용하지 않으면 조인 테이블 방식이 사용되므로 반드시 
    @JoinColumn을 사용하여야 함

    연관관계의 주인이 관리하는 외래키가 반대편 테이블에 존재하므로 연관관계 관리를 위해서는
    insert문으로 한번에 처리되지 못하고 update문을 한번 더 전송하여야 하는 비용이 발생함
    또한, 직접 다루어진 엔티티가 아닌 반대편의 엔티티에 update가 이루어지므로 운영 시에 혼란을 야기함

    따라서, 될 수 있으면 일대다 단방향 연관관계보다는 다대일 양방향 연관관계를 사용하는 것이 더 좋은 방법임

#### 양방향
    
    공식적으로 존재하는 연관관계가 아님

    @JoinColumn(insertable=false, updatable=false)를 이용하여 읽기 전용 필드로
    만들어 마치 양방향인 것 처럼 사용하는 방법임

    좋은 방법이 아니므로 다대일 양방향 연관관계를 사용할 것
    
### 일대일(1:1)

#### 단방향

    서로 일대일 연관관계이므로 외래키가 어느 테이블에 있어도 상관없음
    또한, 일대일 대응이기 때문에 외래 키에 유니크 제약조건이 추가됨

#### 양방향

    다대일 양방향 연관관계처럼 외래 키가 있는 곳이 연관관계의 주인으로써
    반대편에는 mappedBy를 적용하여야 함

#### 외래 키 위치

    주 테이블, 대상 테이블이란 개념 상으로 주체를 가지는 테이블을 말함(Member : 주, Order : 대상)

    주 테이블에 외래키를 두는 경우(객체지향 개발자 선호)

        장점 : 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인 가능
        단점 : 값이 없으면 외래 키에 null 허용
    
    대상 테이블에 외래키를 두는 경우(데이터베이스 개발자 선호)
    
        장점 : 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때 테이블 구조 유지
        단점 : 프록시 기능의 한계로 지연 로딩으로 설정해도 항상 즉시 로딩됨

### 다대다(N:M)

    다대다 연관관계는 실무에서 사용하지 않음

    객체 관점에서는 컬렉션을 사용하여 객체 2개로 다대다 관계가 가능하지만
    관계형 데이터베이스의 경우에는 테이블 2개로 다대다 관계를 표현할 수 없음
    따라서, 다대다 관계를 관계형 데이터베이스는 연결 테이블을 두어 해결함

    문제는 일반적으로 두 테이블을 연결해주는 테이블이 단순히 연결 역할만 하고 끝나지 않음
    각각 테이블의 외래키 두개를 하나의 PK로 만드는데 그치지 않고 추가적으로 필요한
    필드들이 추가되는 경우가 대부분이나 객체를 이용해서는 추가되는 필드들을 컨트롤할 수 있는
    방법이 없음

    다대다 연관관계는 연결테이블을 엔티티로 승격시키고 각각 테이블의 외래키 두개를 하나의 PK로
    관리하지 않고 의미없는 PK를 따로 두어 관리하는 것이 좋음
    PK가 각각의 테이블에 종속적으로 엮이는 순간 시스템이 변화할 때 대응하기가 쉽지 않음