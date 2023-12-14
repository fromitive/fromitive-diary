---
title: "그냥 시켜라"
description: "디미터의 법칙이 무슨 뜻인지 알 것 같아"
time: 2023-12-14 19:23:03
tags:
  - java
  - software-engineering
---

## 프로그램의 설계 품질을 좌우하는 것

은 바로, 객체들간의 최소한의 메시지 전송이라고 한다. 한 객체에서 다른 객체에 메시지를 전송할 때, 다른 객체의 내부 구조를 최대한 숨기는 것이 바로 유연하고, 유지보수, 설계품질을 향상시킨다고 한다.


## 디미터의 법칙

디미터의 법칙은 `낯선자와 대화하지 마라`이다. 인접한 객체끼리는 이야기 하되, 친구의 친구에게 메시지를 보내지 않는다면 설계품질을 향상시킬 수 있다고 한다. 친구 관계도, 주변 친구들과 이야기하면서 부탁하는 건 편한데, 친구의 지인에게 직접적으로 부탁하기에는 어색한 상황을 떠올려보자.


## 묻지 말고 시켜라

이번 글을 작성한 이유이다. 내가 코드를 짤때 보통 boolean 타입으로 `is***()`메소드를 남기곤 하는데, 이렇게 내부 구조를 그대로 들어내서 얻는 장점보다, 코드의 내부구조까지 다 알아야 하는, 단점이 있다면, 지양해야할 메세지라고 한다. **그냥  시켜고 알아서 판단하게 설계**한다면, 코드의 결합도를 낮출 수 있다고 말한다. 제어문도 없어지고, 코드는 말 그대로 알아서 판단할 것이다. 필자가 작성한 버스 정류장 어플리케이션의 상황을 가지고 한 번 리펙터링 해보자.

### Bus의 다음 행선지 리펙터링 하기

Bus 클래스는 아래와 같이 구성되어 있다. `step()`은 `route`에 있는 시작 지점과, `currentNode`를 둘 다 사용하고 있어서 BusNode와 Route와의 의존성이 높은 코드로 보여진다. `currentNode`를 이용해 현재의 위치에 대한 상태 값을 저장하고 있고, 현재 위치를 기반으로 버스 도착시간을 계산하고 있다.

``` java title="Bus.java"
public class Bus {
    private Route route;
    private BusNode currentNode;
    private CrowdLevel crowdlevel = CrowdLevel.FINE;

    public Bus(Route route) {
        this.route = route;
        this.currentNode = route.getStart();
    }

    public void step() {
        currentNode = currentNode.next();
        if (currentNode.isEnd()) {
            currentNode = route.getStart();
        }
    }

    public BusArrival arrivalInfo(BusStop to) {
        return BusArrival.create(crowdlevel, currentNode.arrivalTime(to));
    }
}
```

step() 메시지를 리팩터링 하기 위해 currentNode에게 묻지 말고, 그냥 시키는 형태로 아래와 같이 변경하였다. 그 결과, Route 정보를 알 필요가 없고, 정류장이 종점에 다가오는지 않오는지 신경쓰지 않도록 변경이 되었다. 즉, Bus안에서 변경이 쉬운 코드가 되었다.

``` java title="리팩터링 된 Bus 클래스" hl_lines="5 10"
public class Bus {
    private BusNode currentNode;
    private CrowdLevel crowdlevel = CrowdLevel.FINE;

    public Bus(BusNode node) {
        this.currentNode = node;
    }

    public void step() {
        currentNode = currentNode.next();
    }

    public void setCrowdLevel(CrowdLevel level) {
        this.crowdlevel = level;
    }

    public BusArrival arrivalInfo(BusStop to) {
        return BusArrival.create(crowdlevel, currentNode.arrivalTime(to));
    }
}
```

그럼 우리가 해야 하는 부분은 무엇인가? 바로, Route에서 생성 역할을 맡은 Bus의 생성 방법을 변경하고, BusNode의 내부 로직만 변경하면 된다.

Route는 내부변수인 start를 가지고 있으며, 이를 Bus에게만 넘기도록 변경되었다.

``` java title="Route-변경.java" hl_lines="5"
public class Route {
    private BusNode start;

    public Bus createBus() {
        Bus newBus = new Bus(start);
        return newBus;
    }
}
```

BusNode는 어떠한가?, 노선을 만들 때, 첫 시작지점의 정보를 계속 갖고 있도록 변경하고, next가 null일 경우 start 정보로 덮어 쓰도록 변경하면 된다. 변경할 때 중점적으로 봐야 하는 부분은 `Route`와 `Bus`가 변경할 필요가 없다는 것이다. 클라이언트가 **굳이 내부정보를 알지 못하도록 일단 시킴으로써** 좀 더 내부구조를 변경하기 쉽도록 설계가 되었다고 생각한다.

``` java title="BusNode-변경.java" hl_lines="2 5 15 26"
public class BusNode {
    private BusNode start;

    private BusNode(BusStop current) {
        this.start = null;
    }

    public void addNext(BusNode next, Time time) {
        BusNode addTo = this;
        while (addTo.hasNext()) {
            addTo = addTo.next;
        }
        addTo.next = next;
        addTo.time = time;
        next.start = this;
    }

    private boolean hasNext() {
        return next != null;
    }

    public BusNode next() {
        if (hasNext()) {
            return next;
        }
        return this.start;
    }
}
```

비록, Bus의 step()을 변경할 때, Route와 BusNode를 변경하였지만, BusNode에 대해서는 next()만 알면 되도록 변경이 되었고, 좀 더 이해하기 쉬운 코드로 변경 되었다고 생각한다.
