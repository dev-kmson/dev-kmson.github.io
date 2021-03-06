---
sort: 3
---

# 역할, 책임, 협력

**협력(Collaboration)**:  
객체들이 애플리케이션의 기능을 구현하기 위해 수행하는 상호작용  
**책임(Responsibility)**:  
객체가 협력에 참여하기 위해 수행하는 로직  
**역할(Role)**:  
객체들이 협력 안에서 수행하는 책임들의 집합

---

## 협력

### 행동과 협력

    - 협력: 객체의 행동을 결정, 행동: 객체의 상태를 결정
    - 객체가 참여하는 협력이 객체를 구성하는 행동과 상태를 모두 결정함

## 책임

### 메시지와 책임

    - 책임은 메시지보다 추상적이고 개념적으로 더 큼
    - 적절한 책임의 할당이 설계의 전체적인 품질을 결정함
    - 객체가 메시지를 선택하는 것이 아니라 메시지가 객체를 선택하게 해야함
        - 최소한의 인터페이스, 추상적인 인터페이스가 가능해짐

## 역할

### 객체와 역할

    - 역할을 통하지 않고 객체에 책임을 할당하면 같은 역할을 하는 동일한 책임이 여러 객체에 분산될 수 있음
    - 역할 구현: 인터페이스, 추상 클래스 사용
        - 인터페이스, 추상 클래스: 구체 클래스들이 따라야 하는 책임의 집합을 서술한 것
    - 역할은 객체를 추상화해서 객체 자체가 아닌 협력에 초점을 맞출 수 있게 됨
    - 협력에 적합한 책임을 수행하는 대상이 한 종류라면 객체로 간주, 여러 종류의 객체들이 참여할 수 있다면 역할 

***<span style="color:#f08080">
메시지에 협력하기 위해 객체에 책임을 할당하자   
책임은 객체의 퍼블릭 메서드를 의미  
책임은 곧 객체의 행동이 되고 행동은 객체의 상태를 결정한다  
<br>
동일한 책임이 여러 객채에 분산되지 않도록 역할 즉,  
추상 클래스와 인터페이스를 이용하여 각 객체들이 동일한 책임을 수행할 수 있도록 하자  
</span>***