---
title: "디자인 패턴 - 전략 패턴"
description: "바뀌는 것과 바뀌지 않는 것들을 구분하기"
time: 2023-11-03 18:11:45
tags:
  - java
  - software-engineering
  - design-pattern
---

## 문제

```
SimDuck이라는 회사가 있다. 지금까지 Duck을 상속받은 다양한 Duck들을 구현했다. 이때 사장님이 `날기` 기능을 추가하고 싶을 때, 변경이 적고 확장하기 편하도록 만들기 위해 어떻게 해야 할까?
```

## 문제 1: fly() 메소드를 Duck에 그냥 만들면 어떤 이슈가 발생할까?

``` java title="fly() 메소드 그냥 구현"
class Duck{
    void fly() {
        // 날기 코드
    }
}

class RubberDuck extends Duck{
    @override
    void fly() {
        //날지 못해요 ㅜ 1
    }
}

class WoodDuck extends Duck{
    @override
    void fly() {
        //날지 못해요 ㅜ 2
    }
}
```

1. 날지 못하는 오리 (고무 오리, 나무 오리) 등이 있을 것이다.
2. 날지 못하는 오리들이 생길 때마다 fly()를 변경해야 한다.
3. 날지 못하는 행위에 대해서 유지보수가 어려워 진다.

## 문제 2: fly()를 인터페이스화 시키면 어떤 이슈가 발생할까?

``` java title="fly를 인터페이스로 받도록 함"
class Duck {
}

class RubberDuck extends Duck {
    // fly 구현 안해도 되겠다!1
}

class WoodDuck extends Duck {
    // fly 구현 안해도 되겠다!2
}

class MallardDuck extends Duck implements Flyable {
    @override
    void fly() {
        // fly method 구현1
    }
}

class RedheadDuck extends Duck implements Flyable {
    @override
    void fly() {
        // fly method 구현2
    }
}
```

1. 날지 못하는 새들에 대해선 구현 안해도 되지만, 나는 새들은 구현해야 함.
2. 다만, fly() 가 같은 기능이라면 코드의 중복이 있음. 상속 받는 클래스가 점점 많아진다면 같은 기능을 수행하는 fly()는 복붙해야 하는 이슈가 있음


## 디자인 원칙1: 애플리케이션에서 달라지는 부분을 찾아내고 달라지지 않는 부분과 분리한다.

{==**일단, 눈에 보이는 변화하는 것 부터 생각하자**==} Duck은 안변할 것 같고, RubberDuck 등등도 마찬가지 이다. 하지만, fly는 날개로 날거나, 로켓으로 날거나, 아예 날지 않는 내용으로도 넣을 수 있다.

이렇게 변화하는 것들을 `캡슐화` 한다.

캡슐화란, fly()를 호출하지만 내부 로직은 모르는 채로 사용하게 되는 것이라고 생각하면 된다. 같은 fly()를 수행하지만, FlyWithWings과 FlyNoWay의 내부로직은 외부로 부터 제어할 수 없다.

``` java title="캡슐화 된 나는 행동"
interface FlyBehavior{
    void fly(); //나는 행동
}

class FlyWithWings implements FlyBehavior {
    @override
    void fly(){
        // 날개를 가지고 나는 행동
    }
}

class FlyNoWay implements FlyBehavior {
    @override
    void fly(){
        // 날지 않음
    }
}
```

## 디자인 원칙2: 구현보다는 인터페이스에 맞춰서 프로그래밍 한다.

> 핵심은, 실제 실행 시에 쓰이는 객체가 코드에 고정되지 않도록 상위 형식에 맞춰 프로그래밍해서 다형성을 활용해야 한다는 점에 있습니다.

인터페이스 또는 추상 클래스를 사용하게되면 {==**행동이 무엇인지 몰라도**==}, 다른 외부 도메인이 사용할 수 있게 된다.

## 디자인 원칙3: 상속보다는 Composition을 사용한다. 
나는 행동에 대한 맴버를 추가하여 flyBehvior를 통해 생성자를 이용하여 나는 행위를 처음부터 지정할 수 있거나, 유연하게 나는 행위를 변경할 수 있음(날지 못했지만 로켓으로 날도록 변경하거나) 
``` java title="Composition 예"
class Duck {
    FlyBehavior flyBehavior; // composition 예
    
    void performFly(){
        flyBehavior.fly();
    }
}
```

