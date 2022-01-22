---
sort: 8
---

# OrphanRemoval

---

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
