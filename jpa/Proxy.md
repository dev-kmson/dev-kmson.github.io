---
sort: 6
---

# Proxy

---

    연관관계가 정의되어 있는 엔티티를 조회할 때 연관관계 대상이 되는 엔티티를 항상 같이 불러올 필요는 없음
    
    연관관계 대상 엔티티를 필요로 할 때 조회하도록 하기 위해서 JPA는 실제 데이터베이스에서 조회한
    엔티티가 아닌 프록시 엔티티로 대체하며 엔티티를 팔요로 하는 시점에 데이터베이스에서 조회 후
    프록시 엔티티의 target(Entity타입의 변수)에 실제 엔티티를 보관함

    즉, 실제 엔티티를 필요로 하는 요청이 일어나기 전까지는 프록시 엔티티로 대체하여
    매번 연관관계가 맺어진 엔티티를 조회하는 낭비를 막고 실제 엔티티가 필요할 때
    PersistenceContext를 통해 데이터베이스에서 조회하여 실제 엔티티 생성 및
    프록시 엔티티가 참조할 수 있도록 함

    프록시 엔티티는 실제 엔티티를 필요로 할 때 딱 한 번만 초기화(실제 엔티티 참조 연결),
    초기화가 된 이후에는 프록시 엔티티를 통해서 실제 엔티티에 접근 가능함

        초기화된다고 해서 프록시 엔티티가 실제 엔티티로 대체되는 것은 아님

        프록시 엔티티는 실제 인티티를 상속받은 객체이기 때문에 타입 비교 시 instance of를 사용

        PersistenceContext로부터 관리되지 않는 준영속 상태인 경우에는 프록시 초기화 시 오류 발생(LazyInitializationException)

        프록시 엔티티 객체 조회를 위한 em.getReference()를 호출하여도 
        PersistenceContext에 이미 관리되고 있는 엔티티라면 실제 엔티티를 반환함

    em.find() : 실제 엔티티 조회
    em.getReference() : 프록시 엔티티 조회
    emf.getPersistenceUnitUtil.isLoaded(entity) : 프록시 엔티티 초기화 여부 확인
    Hibernate.initialize(entity) : 프록시 엔티티 강제 초기화

## Lazy Loading

    연관관계가 맺어진 엔티티를 실제 필요로 할 때 조회하도록 
    프록시 엔티티로 대체하는 것은 지연 로딩(Lazy Loading)인 경우에만 해당됨

    즉시 로딩(Eager Loading)인 경우에는 실제 필요로 할 때 조회하는 것이 아닌
    엔티티가 초기화될 때 연관관계가 맺어진 엔티티도 함께 초기화함

```java

    @Entity
    public class Member {
    
        @Id @GeneratedValue
        private Long id;
        
        @Column(name = "USERNAME")
        private String name;
   
        //@ManyToOne(fetch = FetchType.EAGER) // 즉시 로딩
        @ManyToOne(fetch = FetchType.LAZY) // 지연 로딩
        @JoinColumn(name = "TEAM_ID")
        private Team team;
        
        // ...
    
    }

```

    즉시 로딩의 문제점 :
    
        즉시 로딩인 경우에는 연관관계가 맺어진 엔티티도 같이 조회하기 위해서
        가능하면 조인을 사용해서 한번에 조회를 시도함

            연관된 엔티티가 많으면 많아질수록 조인되어지는 테이블 또한 많아짐
            여러 테이블을 조회하여 한번에 조회 시 성능 상에 큰 문제 발생

        JPQL 사용 시 연관된 엔티티에 대한 조회 시도가 추가적으로 일어남

            JPQL은 단순 SQL 쿼리처럼 작동하기 때문에 연관된 엔티티에 대한
            정보를 알 수 없으며 조회된 후에야 즉시 로딩인 경우 연관된 엔티티의
            조회가 일어남

    즉시 로딩의 문제점으로 인하여 모든 연관관계에 지연 로딩을 사용하여야 함

        @ManyToOne, @OneToOne -> default : 즉시 로딩
        @OneToMany, @ManyToMany -> default : 지연 로딩

## CASCADE

```java
    
    @Entity
    @Table(name = "ORDERS")
    public class Order {
    
        // ...
        
        @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
        private List<OrderItem> orderItems = new ArrayList<>();
        
        // ...
        
    }

```    
    
    특정 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화하기 위해 사용

    해당 엔티티가 여러 곳에 연관되어 있다면 사용하면 안됨
    영속성 전이(CASCADE)로 인하여 의도치 않은 영속화 또는 삭제가 될 수 있음

## OrphanRemoval

```java
    
    @Entity
    @Table(name = "ORDERS")
    public class Order {
    
        // ...
        
        @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
        private List<OrderItem> orderItems = new ArrayList<>();
        
        // ...
        
    }

```

```java

    Order findOrder = em.find(Order.class, order.getId());
    findOrder.getOrderItems().remove(0); // 연관된 엔티티 중 하나를 컬렉션에서 제거

```

    연관관계가 끊어진 엔티티를 자동으로 삭제할 때 사용

    컬렉션에서 지워진 엔티티는 주엔티티와 연관관계가 끊어진 고아 객체로
    orphanRemoval 옵션이 true라면 해당 엔티티에 대한 delete문이 호출됨

    주엔티티가 지워진 경우 또한 고아 객체가 된 엔티티로 간주되어
    orphanRemoval 옵션이 true라면 해당 엔티티에 대한 모든 delete문이 호출됨

    영속성 전이와 마찬가지로 여러 곳에 연관되어 있는 엔티티라면 사용하면 안됨
    


    



    