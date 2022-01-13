---
sort: 5
---

# MappedSuperclass

---

```java

    @MappedSuperclass
    public abstract class BaseEntity {
    
        private String createdBy;
        private LocalDateTime createdDate;
        private String lastModifiedBy;
        private LocalDateTime lastModifiedDate;
    
        // ...
        
    }

```

```java
    
    @Entity
    @Inheritance(strategy = InheritanceType.SINGLE_TABLE)
    @DiscriminatorColumn
    public abstract class Item extends BaseEntity {
    
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
    
        private String name;
        private int price;
        
        // ...
        
    }

```

    - 클래스 레벨에 사용

    - 모든 객체(테이블)에서 공통적으로 쓰이는 매핑 정보가 필요할 때 사용

    - 공통 매핑 정보를 만들어 두고 공통 매핑 정보가 필요한 엔티티에 상속

        extends를 통해 사용하지만 InheritanceMapping과는 상관 없음

    - 단순 매핑 정보만 제공하며 직접 생성해서 사용할 일이 없으므로 추상 클래스로 만드는 것을 권장함
    