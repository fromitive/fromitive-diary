---
title: "버스 정류장 - 이전보다 나아진 버전?"
description: "실제 있는 사물 외에 무언가가 더 있다..?"
time: 2023-12-11 21:52:45
tags:
  - java
  - software-engineering
---

## 두 번째 버스 정류장

버스정류장의 다음 버전을 만들게 되었다. 이전 버전에서는 아래와 같이 `버스 정류장`과 `노선`이 강한 결합을 가지고 있어서 이를 유지할까 고민하였다.

``` java title="기존 코드" hl_lines="3 5 9 18"
public class BusStop {
    public void addDisplay(Display display) 
    private Map<Route, Time> getArraiveTime()
    public void showArriveTime()
    public void addRoute(Route route)
}

public class Route {
    public Route(String name, BusStop busstop) {
        BusNode newNode = new BusNode(busstop);
        this.start = newNode;
        this.end = newNode;
        this.name = name;
        busstop.addRoute(this);
    }
    public BusNode getStart()
    public void addBus(Bus bus)
    public void addNext(BusStop busstop, Time arrival)
    public Time arrivalTime(BusStop busstop)
    public void displayRoute()
}
```

`BusStop`과 `Route`가 생성하거나, 경로를 추가할 때 강한 의존성을 가지고 있어서 둘 중 한 곳이 수정될 때, 다른 한쪽이 수정될 수 밖에 없도록 설계가 되어있었다. 또한, 모든 버스 정류장이 생성한 경로를 다 갖고있는것이 아니라 둘의 관계는 **N:M**의 관계를 가지고 있다. 이런 관계를 유지할 경우, 미래에 요구사항이 변경될 때 걸림돌이 될 수 있는 확률이 높아진다고 한다.

두 번째 버스노선은 좀 더 다르게 생각하였다. 일단, 버스 노선과, 버스 정류장의 관계를 직접적으로 관련있는 것이 아니라, **매니저를 두어 관리를 하는 방향으로 고민을 해보았다.** 고민한 코드는 아래와 같다.

``` java title="개선된 버전"
class BusStopRouteManager {
    public static void arrival(BusStop arrivalTo) {
    }
}


public class BusStop {
    public void display(ArrivalInfo info) {
    }
}

public class Route {
    BusArrival arrival(BusStop To) {
    }
}
```

버스 정류장의 예상 도착정보를 구할 때는 자연스럽게 매니저를 통해서 물어보는 방향으로 전환하게 되었다. 원하는 버스 정류장의 도착시간을 구할 때 매니저에게 요청을 하여 노선에 대한 직접적인 의존성을 갖지 못하도록 하였다.

매니저가 필요한 정보를 `ArrivalInfo`를 세팅하게 되면 BusStop안에 등록되어 있는 display 메시지를 통해 해당 버스 정류장의 도착 시간을 출력하도록 설계하였다.

`ArrivalInfo`를 만들기 위해 매니저는 등록된 `Route`에 arrival 메시지를 발생시켜 `Route`과 관련된 정보들을 쉽게 변경하기 위해 별도로 `BusArrival`를 통해 정보로 전달해주는 역할을 한다.

## 책임은 그 정보가 가장 많을 것 같은 것이 가져간다

기존 버스 정류장의 문제는 노선도 애매하게 알았고, 노선도 마찬가지로 버스 정류장에 대한 정보를 자세히 알지 못하였다.

이렇게 애매한 상황에서 매니저라는 것을 떠올리게 되었고, **매니저 자체는 버스 정류장과 노선에 대해 잘 아니 도착 정보를 구할 때는 매니저한테 물어보는 것이 가장 적절하다고 자연스레 떠오르게 되었다** 즉, 책임은 정보 전문가에게 놓는다는 규칙을 지킬 수 있게 되었다.

> 메시지를 수신할 적합한 객체는 누구인가?

> 객체는 자신의 상태를 스스로 처리하는 자율적인 존재여야 한다. 객체의 책임과 책임을 수행하는데 필요한 상태는 동일한 객체 안에 존재해야 한다. 따라서 객체에게 책임을 할당하는 첫 번째 원칙은 책임을 수행할 정보를 알고 있는 객체에게 책임을 항당하는 것이다. 이를 INFORMATION EXPERT 패턴이라고 부른다. -오브젝트 139p-


그 결과, 노선과 정류장의 의존 관계가 정리가 되었다. 매니저는 버스 정류장과, 노선을 알고 있지만, 버스 정류장과 노선은 매니저에 대해 잘 모르니, 의존관계과 확실하다. 만일 매니저가 없거나 변경되더라도, 버스와 노선은 변경하지 않아도 상관없게 된다.

## 생성 책임 대한 고민

버스정류장을 기반으로 버스 노선을 만들게 된다. 매니저는 버스 정류장과 버스 노선의 전문가로서, 버스 정류장을 바탕으로 버스 노선을 만들 수 있는 권한을 주어도 된다고 생각한다. 그리고 이러한 생각은 노선과 버스 정류장이 서로의 의존성을 약화시키도록 만들 수 있게 한다. 객체를 보통 자기 자신이 생성하도록 하지만, 매니저가 노선을 생성하도록 통제한다면 좀 더 유연하게 코드를 짤 수 있을 것 같다.

따라서, 매니저에 `Builder` 패턴을 적용하여 버스 정류장을 기반으로 노선을 생성하면 이를 관리하도록 책임을 할당하였다. 매니저가 노선을 관리한다면 그리고, 노선이 버스 정류장을 직접적으로 관리하지 않고 매니저를 통해 관리하게 된다면, 프로그램의 노선과 정류장의 관계를 한 눈에 파악할 수 있고, 데이터를 중앙에서 관리할 수 있는 장점이 있다고 생각한다.

``` java title="BusStopRouteManager.java"
class BusStopRouteManager {

    public static class RouteBuilder {
        private RouteBuilder(String routeName, BusStop start) {
        }

        public static RouteBuilder create(String routeName, BusStop start) {
        }

        public RouteBuilder nextRoute(BusStop next, Time time) {
        }

        public Route build() {
        }
    }
}
```

이번엔 버스에 대해 고민을 한다. 버스는 버스 노선을 따라 이동한다. 그러므로 버스 노선쪽에서 버스를 생성하는 책임을 할당하게 되면 좋을 것 같아 아래와 같이 버스를 생성하게 하였다. 이렇게 적으니 노선은 버스의 매니저의 역할을 가진 것 처럼 보인다.

``` java title="Bus.java"
public class Route {
    public Bus createBus() {
    }
}

```

<<오브젝트에서>> 객체를 생성해야 할 때 아래와 같은 대상에 대해 책임을 할당하라고 한다. 매니저는 버스 노선에 대한 기록을 가지고 있어야 했고, 버스 노선도 마찬가지로 버스가 현재 어느 위치에 있는지 파악하기 위해 기록을 해야하기 때문에 생성에 대한 책임을 버스 노선에 할당하는 것이 적합해 보였다. 

> CREATOR 패턴

> 아래의 조건을 만족하는 B에게 생성 책임을 할당하라

> B가 A 객체를 포함하거나 참조한다.

> B가 A 객체를 기록한다.

> B가 A 객체를 긴밀하게 사용한다.

> B가 A 객체를 초기화하는 데 필요한 데이터를 가지고 있다(B는 A에 대한 정보 전문가다)

> -오브젝트 145p-

## 예외 상황

버스 노선과 버스 정류장 그리고 버스와의 관계를 정리하였다. 올바른 결과물이 나타나기 전, 아래와 같은 예외가 발생할 경우 어디서 고려해야 하는지 고민할 필요가 있다.

```
1. ArrivalInfo에 아무런 정보가 없을 경우 어떻게 처리하지?

2. 버스가 다니고 있지 않을 경우 빈 값을 어떻게 나타낼 것인가?
```

이를 바탕으로 나는 정보가 없을 경우 어떻게 처리해야 할지 책이나 bestpractice를 통해 고민해야 할 것 같다. 그에 대한 내용은 **Optional 사용이나, 객체에서 빈 값에 대해 별도로 정의하는 정도라고 생각하니, 다음 공부 목표는 이러한 빈 값 처리나, Null을 어떻게 처리할 것인지에 대해 조사해봐야겠다.**
