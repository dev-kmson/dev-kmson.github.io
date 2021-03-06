---
sort: 8
---

# 의존성 관리하기

---

## 의존성 이해하기

### 변경과 의존성

    - 두 요소 사이의 의존성이 존재한다는 것은 의존되는 요소가 변경될 때 함께 변경 될 수 있음을 의미

### 의존성 전이

    - A -> B -> C
    - A가 의존하는 B가 C를 의존하므로 B의 의존성이 A에 전이되었음  
    - 의존성 전이(transitive dependency): 한 요소가 다른 요소에 의존할 때 그 대상 요소가 의존하는 다른 요소 또한 의존하게 되는 것  

    - A는 B를 직접 의존, B는 C를 직접 의존  
    - 직접 의존성(direct dependency): 한 요소가 다른 요소에 직접 의존하는 경우  

    - A는 C를 간접 의존  
    - 간접 의존성(indirect depency): 직접적인 관계는 없지만 의존성 전이에 의해 영향을 받는 경우

    - 의존성은 전이될 수 있으나 내부 구현을 효과적으로 캡슐화 했다면 전이되지 않을 수 있음

### 런타임 의존성과 컴파일타임 의존성

    - 컴파일 시점에 가지는 의존성과 런타임 시점에 가지는 의존성이 다름  

    - 컴파일 시점: 다형성을 이용하여 클라이언트 객체는 인터페이스만을 바라봄  
    - 런타임 시점: 인터페이스 타입에 대입된 구현체가 실제 의존을 가지는 객체가 됨

### 컨텍스트 독립성

    - 인터페이스가 아닌 구현체를 의존하는 경우 해당 구현체의 문맥에 강하게 결합됨  
    - 강하게 결합된다는 것은 해당 구현체만을 위한 클래스가 되기 쉬움

    - 구현체의 문맥에 얽매이지 않는다 -> 컨텍스트에 독립적이다

    - 컨텍스트 독립성: 구현체의 문맥에 얽매이지 않는 정도

### 의존성 해결하기

    - 의존성 해결: 컴파일타임 의존성을 실행 컨텍스트에 맞는 적절한 런타임 의존성으로 교체하는 것  
    - default로 적용할 구현체는 생성자를 통해 주입시키고, 이후 필요에 따라 setter 메서드를 통해 의존 대상 변경하는 것이 가장 선호되는 방법

    - 객체를 생성하는 시점에 생성자를 통해 의존성 해결
    - 객체 생성 후 setter 메서드를 통해 의존성 해결
    - 메서드 실행 시 인자를 이용해 의존성 해결

    - 구현체에 직접 의존하지 않고 외부로부터 의존성을 주입받아 특정 구현체에 결합되지 않도록 함을 의미 

## 유연한 설계

### 의존성과 결합도

    - 바람직한 의존성은 재사용성과 관련
    - 컨텍스트에 독립적인 의존성은 바람직한 의존성, 특정 컨텍스트에 강하게 결합된 의존성은 바람지하지 못한 의존성

    - 의존성은 두 요소 사이의 관계 유무를 설명
        - 의존성이 존재한다, 의존성이 존재하지 않는다
    - 결합도는 두 요소 사이에 존재하는 의존성의 정도를 상대적으로 표현
        - 결합도가 강하다, 결합도가 느슨하다

### 추상화 의존

    - 인터페이스에 의존하면 상속 계층을 모르더라도 협력이 가능해짐
    - 인터페이스 의존성은 협력하는 객체가 어떤 메시지를 수신할 수 있는지에 대한 지식만을 남김
    - 결합도를 느슨하게 만들기 위해선 구체 클래스보단 추상 클래스에, 추상 클래스보단 인터페이스에 의존하게 만들어야 함
    - 의존하는 대상이 더 추상적일수록 결합도는 더 낮아짐
    
    - 인터페이스에 의존 -> 알아야 하는 정보가 줄어듬 -> 컨텍스트에 독립적이 됨 -> 결합도가 낮아짐
                -> 다양한 클래스 상속 계층에 속한 객체들이 동일한 메시지를 수신할 수 있도록 컨텍스트 확장이 가능해짐

### 명시적인 의존성

    - 의존성은 명시적으로 널리 알려야 함

    - 명시적인 의존성: 퍼블릭 인터페이스에 의존성을 드러내어 노출시킴
        - 생성자를 통해 특정 객체를 요구함을 드러내는 경우
    - 숨겨진 의존성: 내부 구현에서 의존 관계가 성립되어 퍼블릭 인터페이스에 노출되지 않음
        - 메서드 내부에서 특정 객체의 인스턴스를 생성하는 경우

### new연산자의 문제점

    - new 연산자를 사용하기 위해서는 구체 클래스의 이름을 직접 기술해야 함
        - 클라이언트는 추상화가 아닌 구체 클래스에 의존할 수 밖에 없음 -> 결합도가 높아짐
    - new 연산자는 생성하려는 구체 클래스뿐만 아니라 생성자 호출 시 어떤 인자가 필요한지도 알고 있어야 함
        - 클라이언트가 알아야 하는 지식의 양이 늘어남 -> 결합도가 높아짐

    - 해결 방법: 인스턴스를 생성하는 로직과 생성된 인스턴스를 사용하는 로직을 분리
        - 인스턴스를 필요로하는 클라이언트가 생성에 관한 정보를 알지 못해도 되므로 결합도가 증가하는 것을 방지

    - 기본적으로 사용하는 타입이 정해져있는 경우에는 내부에서 인스턴스를 생성해도 무방

        - 오버라이딩을 통해 타입 지정하지 않아도 되는 생성자 1개, 타입 지정하는 생성자 1개 만듦 
        - 기본 타입 사용하는 경우 타입 지정하지 않아도 되는 생성자 이용
                -> 생성자에서 기본 타입을 지정하여 타입 지정을 필요로하는 생성자를 다시 호출
        - 다른 타입을 필요로 하는 경우 타입 지정하는 생성자 이용

        - 비록 구체 클래스에 의존하게 되지만 클래스의 사용성이 증가하므로 트레이드오프를 통해 더 나은 대안을 선택

***<span style="color:#f08080">
가능하면 구체 클래스가 아닌 인터페이스에 의존하자  
인스턴스의 사용과 생성의 책임을 분리하자  
의존성을 생성자에 명시적으로 드러내자
</span>***






