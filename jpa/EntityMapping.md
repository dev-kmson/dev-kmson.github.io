---
sort: 2
---

# EntityMapping

---

## hibernate.hbm2ddl.auto

    - persistence.xml에 작성되는 옵션으로 스키마 생성에 대한 처리를 함

```xml
    <property name="hibernate.hbm2ddl.auto" value="create" />
```        

        create : 어플리케이션 기동 시 @Entity로 매핑된 테이블을 DROP 후 CREATE

        create-drop : create와 동일하나 어플리케이션 종료 시점에 DROP

        update : 추가로 작성된 내용만을 ALTER TABLE(추가만 가능하고 삭제는 되지 않음)

        validate : 엔티티와 테이블이 정상 매핑되었는지 확인

        *** 운영에서는 심각한 문제를 초래할 수 있으므로 개발 시에만 이용

## @ Entity

    - 클래스 레벨에 사용

    - @Entity 사용 시 JPA가 관리 대상이 됨

    - public 또는 protected 접근제어자인 기본 생성자 필수

    - final, enum, interface, inner 클래스에 사용 불가

    - 엔티티로 이용되어야 할 필드라면 final 사용 불가

## @ Table

    - 클래스 레벨에 사용
    
    - name 속성을 이용하여 매핑할 테이블 이름 지정 (기본 값은 엔티티 이름 사용)

    - catalog 속성을 이용하여 데이터베이스 catalog 매핑 가능
    
    - schema 속성을 이용하여 데이터베이스 schema 매핑 가능

    - uniqueConstraints 속성을 이용하여 유니크 제약 조건 생성 가능

## 컬럼 매핑

    - 변수 레벨에 사용

        @Column : 매핑할 컬럼의 이름, DDL 자동 생성 옵션 부여 가능

            name 속성을 이용하여 필드와 매핑할 테이블의 컬럼 이름
    
            insertable, updatable 속성을 이용하여 등록, 변경 가능 여부

            nullable 속성을 이용하여 null 값 허용 여부

            unique 속성을 이용하여 유니크 제약조건 부여 가능
            
            columnDefinition 속성을 이용하여 컬럼 정보 직접 기입 가능
                    -> varchar(100) default 'EMPTY'

            length 속성을 이용하여 문자 길이 부여 가능

            precision,scale 속성을 이용하여 BigDecimal 타입 시 전체 자릿수, 소수 자릿수 부여 가능

        @Temporal : 날짜 타입 매핑(LocalDate, LocalDateTime 등 JAVA8 날짜, 시간 이용 시에는 필요없음)

            value 속성으로 TemporalType.DATE, TemporalType.TIME, TemporalType.TIMESTAMP 사용 가능

        @Enumerated : enum 타입 매핑
        
            value 속성으로 EnumType.ORDINAL(enum 순서), EnumType.STRING(enum 이름) 사용 가능

            ORDINAL 이용 시 추후에 순번 변경으로 인하여 부여된 순서가 틀어질 수 있으니 STRING을 사용

        @Lob : BLOB, CLOB 매핑
    
            매핑하는 필드 타입이 문자면 CLOB, 이 외에는 BLOB 매핑

        @Transient : 특정 필드를 컬럼에 매핑하지 않음

##  기본 키 매핑

    - @Id : 기본 키 할당

    - @GeneratedValue

        strategy = GenerationType.IDENTITY

            기본 키 생성을 데이터베이스에 위임(AUTO_INCREMENT)

            엔티티에 기본 키의 값을 직접 입력하지 않고 null로 생성하면 persist() 호출 시
            각 데이터베이스마다 기본 키 생성에 대한 기능을 이용해 키 값을 부여함

            데이터베이스에서 기본 키 생성을 위해 일반적으로 사용되는 AUTO_INCREMENT는
            INSERT SQL문이 실행되어야 부여될 ID 값을 알 수 있음
            이는 즉, persist() 시에 1차 캐시에 엔티티를 저장하기 위한 ID 값을
            알 수 없음을 의미함

            이러한 문제점을 해결하기 위해 JPA는 IDENTITY 전략을 취하면
            persist()를 호출하는 시점에 즉시 INSERT SQL을 실행하고 데이터베이스에서
            ID 값을 조회하여 해당 값으로 1차 캐시에 엔티티를 저장함

        strategy = GenerationType.SEQUENCE

```java
    
        @Entity
        @SequenceGenerator(
            name = "MEMBER_SEQ_GENERATOR",
            sequenceName = "MEMBER_SEQ",
            initialValue = 1, allocationSize = 1)
        public class Member {
        
            @Id
            @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "MEMBER_SEQ_GENERATOR")
            private Long id;
        
        }

```
            
            @SequenceGenerator를 통하여 데이터베이스 시퀀스명, 초기값, 증가값 부여 가능

            엔티티에 기본 키의 값을 직접 입력하지 않고 null로 생성하면
            persist() 호출 시 데이터베이스의 SEQUENCE 기능을 이용해 
            순번을 조회하여 얻어온 ID 값으로 1차 캐시에 엔티티를 저장함

            단, persist() 호출 시마다 SEQUENCE를 데이터베이스에서
            직접 조회하기에는 네트워크 비용이 비효율적으로 발생하므로
            이 문제를 해결하기 위해 JPA는 @SequenceGenerator 이용 시
            allocationSize를 50을 기본 값으로 하여 시퀀스 순번을 미리
            확보한 이후에 확보한 숫자 이전까지는 더 이상 SEQUENCE 조회를
            하지않고 엔티티에 직접 순차적으로 순번을 부여하여 네트워크 비용
            낭비를 최소화함

        strategy = GenerationType.TABLE

```java

            @Entity
            @TableGenerator(
                    name = "MEMBER_SEQ_GENERATOR",
                    table = "MY_SEQUENCES",
                    pkColumnValue = "MEMBER_SEQ", allocationSize = 1)
            public class Member {
                
                    @Id
                    @GeneratedValue(strategy = GenerationType.TABLE, generator = "MEMBER_SEQ_GENERATOR")
                    private Long id;
                
            }

```
            
            데이터베이스의 SEQUENCE 기능을 이용하는 것이 아니고
            테이블을 직접 만들어서 마치 SEQUENCE처럼 테이블을 이용하는 것

            모든 데이터베이스에서 일괄적으로 공통되게 적용할 수 있으나
            데이터베이스에서 지원하는 기능(AUTO_INCREMENT, SEQUENCE)을 
            이용하는 것이 아니라서 성능이 떨어짐
            

            

        
            