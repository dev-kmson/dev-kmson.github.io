---
sort: 8
---

# Embedded Type

---

## Embedded Type

### 임베디드 타입 생성

```java
    
    @Embeddable
    @Access(AccessType.FIELD)
    public class Address {
    
        private String city;
        private String street;
        private String zipCode;
    
        public Address() {
        }
    
        public Address(String city, String street, String zipCode) {
            this.city = city;
            this.street = street;
            this.zipCode = zipCode;
        }
    
        public String getCity() {
            return city;
        }
    
        public String getStreet() {
            return street;
        }
    
        public String getZipCode() {
            return zipCode;
        }
    
    }

```

```java
    
    @Entity
    public class Member {
    
        // ...
    
        @Embedded
        private Address homeAddress;
    
        public Address getHomeAddress() {
            return homeAddress;
        }
    
        public void setHomeAddress(Address homeAddress) {
            this.homeAddress = homeAddress;
        }
    
        // ...
        
    }

```

    임베디드 타입을 이용하여 도메인을 추상화할 수 있음
    
    엔티티의 생명주기에 의존하므로 생명주기에 관해 따로 관리해줄 필요가 없음

    엔티티의 속성들 중 도메인 관점에서 공통된 속성을 따로 추상화하는 것일 뿐,
    임베디드 타입으로 만들다고 하여 실제 매핑되는 테이블은 동일함

    도메인 관점에서 추상화 시킨 객체이므로 재사용성을 높이고 응집도가 올라감

### 임베디드 타입 중복

```java
    
    @Entity
    public class Member {
    
        // ...
    
        @Embedded
        private Address homeAddress;
    
        @Embedded
        @AttributeOverrides({
                @AttributeOverride(name = "city", column = @Column(name = "WORK_CITY")),
                @AttributeOverride(name = "street", column = @Column(name = "WORK_STREET")),
                @AttributeOverride(name = "zipCode", column = @Column(name = "WORK_ZIPCODE"))
        })
        private Address workAddress;
    
        public Address getHomeAddress() {
            return homeAddress;
        }
    
        public void setHomeAddress(Address homeAddress) {
            this.homeAddress = homeAddress;
        }
    
        public Address getWorkAddress() {
            return workAddress;
        }
    
        public void setWorkAddress(Address workAddress) {
            this.workAddress = workAddress;
        }
    
        // ...
    
    }

```

    한 엔티티 안에 같은 임베디드 타입을 사용하면 컬럼명 중복으로 인하여 오류가 발생함
    @AttributeOverides, @AttributeOverride 사용하여 컬럼명을 재정의할 수 있음

### 임베디드 타입 비교

```java

    @Embeddable
    @Access(AccessType.FIELD)
    public class Address {
    
        private String city;
        private String street;
        private String zipCode;
    
        public Address() {
        }
    
        public Address(String city, String street, String zipCode) {
            this.city = city;
            this.street = street;
            this.zipCode = zipCode;
        }
    
        public String getCity() {
            return city;
        }
    
        public String getStreet() {
            return street;
        }
        
        public String getZipCode() {
            return zipCode;
        }
    
        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            Address address = (Address) o;
            return Objects.equals(getCity(), address.getCity()) && 
                    Objects.equals(getStreet(), address.getStreet()) && 
                    Objects.equals(getZipCode(), address.getZipCode());
        }
    
        @Override
        public int hashCode() {
            return Objects.hash(getCity(), getStreet(), getZipCode());
        }
    }

```

    임베디드 타입과 같이 직접 정의한 객체의 경우에는 == 으로 비교할 수 없으므로
    equals, hashCode를 재정의하여 값 비교를 할 수 있도록 하여야 함

### 임베디드 타입 주의사항

    임베디드 타입을 사용할 때는 주의해야할 점이 있는데 이는 바로 임베디드 타입도 곧 사용자가 정의한 객체이므로
    참조 공유로 인하여 한 곳에서 임베디드 타입 객체의 속성을 수정하게 되면 참조하고 있는 다른 곳에서도 모두 수정된다는 것임

    따라서, 생성자를 이용하여 인스턴스화할 때 딱 한번 초기화되도록 하고 이후에는 변경될 수 없도록 수정자(setter)를 제공하지 않아야 함

## Embedded Type Collection

### 임베디드 타입 컬렉션 생성

```java

@Entity
public class Member {

    // ...

    @Embedded
    private Address homeAddress;

    @Embedded
    @AttributeOverrides({
            @AttributeOverride(name = "city", column = @Column(name = "WORK_CITY")),
            @AttributeOverride(name = "street", column = @Column(name = "WORK_STREET")),
            @AttributeOverride(name = "zipCode", column = @Column(name = "WORK_ZIPCODE"))
    })
    private Address workAddress;

    @ElementCollection
    @CollectionTable(name = "FAVORITE_FOOD", joinColumns = @JoinColumn(name = "MEMBER_ID"))
    @Column(name = "FOOD_NAME")
    private Set<String> favoriteFoods = new HashSet<>();
    
    @ElementCollection
    @CollectionTable(name = "ADDRESS", joinColumns = @JoinColumn(name = "MEMBER_ID"))
    private List<Address> addressHistory = new ArrayList<>();
    
    public Address getHomeAddress() {
        return homeAddress;
    }

    public void setHomeAddress(Address homeAddress) {
        this.homeAddress = homeAddress;
    }

    public Address getWorkAddress() {
        return workAddress;
    }

    public void setWorkAddress(Address workAddress) {
        this.workAddress = workAddress;
    }

    // ...

}

```

    @ElementCollection, @CollectionTable을 이용하여 기본 타입 혹은 임베디드 타입을
    컬렉션에 저장하고 해당 컬렉션을 데이터베이스에 매핑할 수 있음

    데이터베이스는 기본적으로 컬럼에 컬렉션 형태를 저장할 수 없으므로 
    별도의 컬렉션이 저장될 테이블 생성 필요

### 임베디드 타입 컬렉션 문제점

    엔티티와 다르게 식별자가 존재하지 않아 값이 변경되면 추적이 어려움

    임베디드 타입 컬렉션에 변경 사항이 발생하면, 주인 엔티티와 연관된 모든 데이터를 삭제하고
    임베디드 타입 컬렉션에 저장되어 있는 값을 모두 다시 저장함

    기본 타입 컬렉션, 임베디드 타입 컬렉션과 매핑되는 테이블은 모든 컬럼이 묶여서 기본키로 구성됨

    위의 문제점들로 인하여 기본 타입 컬렉션, 임베디드 타입 컬렉션은 사용해서는 안됨
    기본 타입 컬렉션 또는 임베디드 타입 컬렉션 대신 엔티티로 승격하여 일대다 관계를 이용하는 것을 고려