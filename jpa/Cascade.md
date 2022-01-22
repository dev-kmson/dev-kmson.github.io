---
sort: 7
---

# CASCADE

---

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
