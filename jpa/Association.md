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
    저장된 Id값을 이용하여 Member Entity에 외래키로써 저장하였음

    객체지향적이라면 객체간의 참조를 통하여 연관된 객체를 찾아야하지만
    데이터 지향 모델링으로 설계한 순간 마치 관계형 데이터베이스의 외래키를
    통해 연관된 테이블을 찾는 듯이 동작하고 있믐을 알 수 있음

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


    
    
