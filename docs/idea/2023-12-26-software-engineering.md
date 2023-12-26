---
title: "의존성 역전 원칙, 의존성 주입, 개방 폐쇄 원칙"
description: "캡슐화를 통해 객체의 협력 관계만 남기도록 하는 것일까?"
time: 2023-12-26 16:21:36
tags:
  - software-engineering
---

## 개방 폐쇄 원칙

개방 폐쇄 원칙은 객체의 협력 관계를 변경하지 않으면서도 확장은 유연하게 설계하는 방법이라고 이해했다. 지불 기능을 제공하는 `Payment` 인터페이스가 존재하고, `process` 라는 협력을 제공하는 가정을 들어보자. 리턴 값이 `Money` 객체를 반환하는 것이지만, 사실 Return 값도 상관이 없고 필자가 임의로 지은 것이다.

``` java title="Payment.java"
interface Payment {
    Money process();
}
```

그리고 Payment와 협력하는 `Store` 클래스가 존재한다고 가정하자. Store는 다른 사람에게 물건을 팔때 `Payment`와 협력을 통해 지불하는 기능을 사용하고 있다.

``` java title="Store.java" hl_lines="7"
class Store {

    private Payment payment;

    void sellTo(Client client) {
        // ...(중략)...
        Money money = payment.process();
        // ...(중략)...
    }  
}
```

현재 `Payment`는 인터페이스이기 때문에, `implements`를 통해 여러가지의 지불 방식을 제공할 수 있게 된다. 즉, 협력 관계는 폐쇄적이지만, 다른 지불방식을 추가하기 용이하게 만드는 원칙이 개방 폐쇄 원칙이라고 생각한다. 

`Payment`의 `process`형태가 바뀌지 않는 이상, 확장하기 편해진다.

그러나 만일, `process`가 계속 변하는 것이라면 기존 `Payment`를 통해 확장했던 방식들을 전부 변경해야 하는 단점이 있다.  
따라서 소프트웨어 설계 시 중요한 건, **객체간 변하지 않는 협력 방식이 무엇인지 잘 정의하는 것이 유연하게 확장하는 설계가 아닌지 생각된다.**

## 의존성 역전 원칙

엉클 밥이 이야기 하는 의존성 역전이란 뜻은. 구체화 하지 않은 상위 계층(즉, 부모 객체)에 의존하라는 뜻이라고 생각된다. 필자는 소프트웨어를 설계할 때, 대부분은 추상화 하지 않고, 구체화 하려는 습관을 가지고 있었다.

다시 `Payment`와 `Store`의 예를 살펴보자. 현재 `Store`는 구체적인 방식인 `CashPayment`를 생성하여 지불을 진행하고 있다.

``` java title="Store.java"
class Store {
    private Payment payment = new CashPayment();
}

```

이는, 이미 `CashPayment`에 강하게 결합되어 있고 지불 방식을 변경하기 위해서는 `Store`객체 생성 시, `CashPayment` 부분을 다른 지불 방식 (예 : `CraditPayment`) 로 변경해야 한다. 

이를 해결하기 위해선 `Store`가 생성자를 Payment로 받도록 변경해야 한다. `Payment`는 인터페이스인데, 초기화 가능한 이유는, **java언어 특징인 컴파일 시점의 의존성과, 런타임 시점의 의존성이 다르도록 작성할 수 있기 때문이다**.

즉 아래의 예제 처럼 Store를 생성할 때, 구체화된 클래스를 바탕으로 초기화가 진행되면 런타임 시점에 구체화 된 클래스가 실행된다. 이를 업케스트라고 하고, 객체 지향의 특성인 폴리모피즘(다형성)이라는 특징이라고 생각한다.

``` java title="Store.java"
class Store {
    
    private Payment payment;

    // 컴파일 시점엔 인터페이스인 Payment를 인자로 받아도 오류가 나지 않음
    public Store(Payment payment) { 
        this.payment = payment;
    }
}

// 메인 코드
public static void main(String [] args) {
    Store store = new Store(new CraditPayment()); // 런타임 시점에 Payment는 CraditPayment로 구체화된 기능을 수행
}

```

위와 같이 내부의 협력관계를 유지하고 외부의 코드로 부터 종류가 다른 런타임 의존성을 부여하는 것을 의존성 주입이라고 하며, Store가 CreditPayment를 직접 의존하지 않고 Payment를 통해 의존하는 양상을 의존성 역전 원칙이라고 한다.
