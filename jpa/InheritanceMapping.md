---
sort: 4
---

# InheritanceMapping

---

    - 상속관계 매핑

        객체지향언어에서는 상속 관계를 지원하나 관계형 데이터베이스에서는 상속관계를 지원하지 않음

        관계형 데이터베이스에서는 상속관계와 유사한 논리 모델로 슈퍼타입 서브타입 관계 논리 모델링 기법이 존재함

        상속관계 매핑은 객체의 상속 구조와 관계형 데이터베이스의 슈퍼타입 서브타입 관계 논리 모델을 매핑하는 것을 말함

            슈퍼타입 서브타입 관계 논리 모델은 3가지 구현 전략을 취함(조인 전략, 단일 테이블 전략, 개별 테이블 전략)

## @ Inheritance

    - 클래스 레벨, 슈퍼 클래스에 사용

    - 슈퍼타입 서브타입 관계 논리 모델의 3가지 구현 전략에 대응되는 옵션 제공

        @Inheritance(strategy = InheritanceType.JOINED) : 조인 전략
        @Inheritance(strategy = InheritanceType.SINGLE_TABLE) : 단일 테이블 전략
        @Inheritance(strategy = InheritanceType.TABLE_PER_CLASS) : 개별 테이블 전략

    - 슈퍼 클래스를 직접 생성하여 사용할 일이 없으므로 추상 클래스로 만드는 것을 권장함

## @ DiscriminatorColumn

    - 클래스 레벨, 슈퍼 클래스에 사용

    - 슈퍼 클래스(슈퍼타입 테이블)에서 어떤 서브 클래스(서브타입 테이블)인지 구분할 수 있는 컬럼 제공

        name 속성을 통하여 컬럼명 설정 가능(default : "DTYPE")

## @ DiscriminatorValue

    - 클래스 레벨, 서브 클래스에 사용

    - @DiscriminatorColumn을 통해 지정된 컬럼에 어떤 value를 넣을 것인지 설정

        문자열을 입력하여 값 설정 가능(default : 엔티티명)

## 조인 전략

```java

    @Entity
    @Inheritance(strategy = InheritanceType.JOINED)
    @DiscriminatorColumn
    public abstract class Item {
    
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
    
        private String name;
        private int price;
    
        // ...
        
    }

```

```java

@Entity
@DiscriminatorValue("M")
public class Movie extends Item {

    private String director;
    private String actor;
    
    // ...
    
}

```

    - 서브타입 테이블들의 공통된 컬럼을 슈퍼타입 테이블에 표현

    - 장점

        테이블 정규화가 잘 이루어진 구현 전략

        외래 키 참조 무결성 제약조건 활용 가능

        서브타입 테이블들의 공통된 속성을 슈퍼타입 테이블에 둘 수 있으므로
        서브타입 테이블의 조회없이 슈퍼타입 테이블 조회만으로 어느정도 처리가 가능함

    - 단점

        조회 시 기본적으로 조인이 사용되기 때문에 단일 테이블 전략 대비 성능 저하

        데이터 저장 시 별개의 INSERT문이 슈퍼타입 테이블, 서브타입 테이블에 각각 전송되어야 함

## 단일 테이블 전략

```java

    @Entity
    @Inheritance(strategy = InheritanceType.SINGLE_TABLE)
    @DiscriminatorColumn
    public abstract class Item {
    
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
    
        private String name;
        private int price;
    
        // ...
        
    }

```

    - 서브타입 테이블들을 슈퍼타입 테이블 하나로 표현

    - 서브타입 테이블들이 테이블 하나로 표현되므로 @DiscriminatorColumn 필수

    - 장점

        슈퍼타입 테이블에 서브타입 테이블 데이터들이 모두 들어가 있으므로 조인이 필요하지 않음

            단일 테이블 하나만 조회하면 되기 때문에 조회 성능이 빠름

    - 단점

        특정 서브타입 테이블에 해당하는 컬럼 이외의 다른 컬럼은 모두 null 데이터가 들어감

## 개별 테이블 전략

```java

    @Entity
    @Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
    public abstract class Item {
    
        @Id
        @GeneratedValue(strategy = GenerationType.TABLE)
        private Long id;
    
        private String name;
        private int price;
    
        // ...
        
    }

```
    - 슈퍼타입 테이블 없이 서브타입 테이블들을 개별적으로 표현

    - 조인 전략과 단일 태이블 전략과는 다르게 개별적으로 테이블이 분리되어 있으므로 @DiscriminatorColumn 불필요

    - 개별 테이블 전략은 다른 전략과는 다르게 @GeneratedValue의 strategy 옵션으로 IDENTITY 사용 불가능

    - 장점

        각각의 서브타입 테이블 별로 데이터가 생성되기 때문에 not null 제약조건 사용 가능

    - 단점

        슈퍼 클래스를 통하여 서브타입 클래스들의 데이터를 조회할 때 매핑되는 모든 서브타입 테이블들의
        데이터를 UNION을 통하여 조회하여야 하는 치명적인 단점 존재

## 전략 선택 방법

    복잡한 구조를 가져 정규화가 잘 이루어져야 한다면 조인 전략
    
    단순하고 확장 가능성이 없다고 판단될 경우 단일 테이블 전략

    개별 테이블 전략은 사용하지 말 것

