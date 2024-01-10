---
title: "상속의 올바르지 않은 예제들"
description: "클라이언트 입장에서 행동이 호환 가능한지 생각하자"
time: 2024-01-10 17:33:47
tags:
  - software-engineering
---

## 개요

코드를 재사용하거나, 타입 계층을 구성하기 위해서 상속을 사용한다고 한다. 상속은 부모의 메시지를 넘어 자식에게 새로운 메시지를 주기위함 보다는, 부모의 메시지를 호환 가능하도록 사용하는 것이 재사용의 의미로 적절하다고 생각한다. **여기서 코드의 재사용의 오해할 수 있는 부분은 자식의 관점에서 재사용하는 것이 아닌, 부모 클래스를 사용하는 클라이언트의 관점에서 재사용하는 것이다**

자식은 부모의 메시지를 오버라이딩 하여 클라이언트는 메시지를 재사용하여 부모의 기능을 확장시키는 관점에서 생각한다면 앞으로 상속을 사용하기 위한 기준을 잡을 수 있을 것이다.  

## 상속의 올바르지 않는 예

아래는 옛날부터 상속의 올바르지 않는 고전적인 예이다. 이를 바탕으로 상속일 사용하지 않는 경우를 숙고하는데 도움이 되길 바란다.

### Vector와 Stack의 관계 

Java에는 util안에 `Stack`을 가지고 있다. `Stack`은 `Vector`의 자식 클래스다. 

`Stack`은 데이터를 추가할 때 `push()` 데이터를 꺼낼 때 `pop()`을 사용한다. 

Stack을 상속받아서 element의 개수를 파악하는 `CountableStack`을 만드는 상황을 가정해보자. 이는 Stack에 넣은 item의 개수를 파악하는 기능을 추가한 Stack의 자식 클래스이다.

`push()`를 오버라이딩 하여 count 맴버변수를 통해 item의 개수를 새는 기능을 추가하면 아래와 같다.

``` java title="CountableStack.java"
class CountableStack<E> extends Stack<E> {
    int count = 0;

    @Override
    public E push(E item) {
        count += 1; // add count
        return super.push(item);
    }

    public int getCount(){
        //get all counts
        return this.count;
    }
    
}

```

java Documentation에서 Stack의 정의를 보면 아래와 같다.

``` java title="Stack.java"
public class Stack<E> extends Vector<E> {
    // ...
}
```

Stack은 Vector를 상속했기 때문에 **Vector의 메시지를 그대로 물려 받는다**. 즉, Stack은 이름만 Stack 이지 `push()` 뿐만 아니라 `add()`메시지를 처리할 수 있게 된다.

CountableStack은 Stack의 자식 클래스이지만 Stack은 Vector의 자식클래스이기 때문에 add 메시지를 사용할 수 있다.

즉, add로 추가한 element는 새지 않기 때문에 개발자의 의도와 다른 결과값이 출력된다.

``` java
jshell> CountableStack<Integer> custom = new CountableStack<>();
custom ==> []

jshell> custom.add(1);
$3 ==> true

jshell> custom.add(2);
$4 ==> true

jshell> custom.getCount(); // expacted 2
$5 ==> 0
```

결론은 Stack의 `push()`의 사례를 볼 수 있듯 **메시지를 확장하기 위해 상속을 사용하는 것은 프로그램의 기능을 예측하기 어렵게 만든다**.


### 새는 날 수 있다. 그런데 팽귄은?

하나의 예제를 더 들어보자. 이번 문제는 상속을 잘 못사용하는 고전적인 대표적 사례이다.

`새`라는 클래스를 아래와 같이 정의할 수 있고, 대부분의 새는 날 수 있다.

``` java title="Bird.java"
abstract class Bird {
    abstract void fly(); // 나는 메시지를 전달하는 코드
}
```

그리고 클라이언트 코드에서 Bird 를 매개변수를 하는 `runWithFly()` 가 다음과 같이 있다고 하자.

``` java title="Client.java"
class Client{ 
    void runWithFly(Bird bird) {
        bird.fly();
    }
}
```

이러한 상황에서 날 수 있는 새인 까마귀, 참새, 그리고 비둘기와 같은 일반적인 새를 가정하였을 때 Bird를 상속 받아 사용할 수 있을 것이다.

그러나 날지 못하는 새인 `팽귄` 및 `키위새`는 현실에서는 새로 분류되어 Bird 클래스를 상속 받을 수 있다고 생각이 되지만, 날지 못하기 때문에 `fly()` 메시지를 전달할 때 어떻게 처리가 될 지 알수 없게 된다.

<figure markdown>
![Image title](/fromitive-diary/assets/2024-01-10/kiwi.png){ width="800"}
<figcaption>그림 - 날지 못하는 키위새(인터넷 검색 ㄱㄱ)</figcaption>
</figure>

또한, `Client` 측에서 Bird는 날 수 있음을 가정하기 때문에 프로그램이 클라이언트의 의도대로 동작되기 어려워 보인다.

이는 **클라이언트의 관점으로 상속을 사용하기 보다는, Bird의 관점에서 생각해서 발생하는 일이라고 생각한다.**

또한, Bird와 같은 사례 처럼 프로그래머가 세운 가정이 틀릴 가능성이 있기 때문에, Bird가 fly()를 지원한다 라고 생각하기 보다는, fly()라는 행동 관점으로 아래와 같은 interface를 구성하는 것이 코드의 확장 관점에서 유리하다고 한다. 이를 인터페이스 분리 원칙이라고 한다. 
``` java title="Flyable.java"
interface Flyable {
    void fly();
}

class Bird {

}

// 참새 클래스
class Sparrow extends Bird implements Flyable{
    @Override
    void fly() {
        // 구현..
    }
}

// 키위새 클래스
class kiwiBird extends Bird {
    // 날지 못한다.
}

```

다시 한 번 노파심에서 말한다. 프로그램을 짤 때 학생, 교사, 새 등등 객체 단위로 생각하곤 하지만, **사용하는 클라이언트와 행동 관점으로도 고려해본다면** 좀 더 확장이 쉽고 유연한 소프트웨어를 만들 수 있다고 생각한다. 

### 정사각형은 직사각형에 포함되나?

위와 같은 예제를 더 알아보자. 수학 세상에는 정사각형은 직사각형의 개념 안에 포함된다. 프로그래밍 세계에도 이 내용이 통할까?

프로그래밍 세계는 사실 정답은 존재하지 않지만, **정사삭형이 직사각형 개념 안에 포함 될 수 없는 가능성**도 생각해봐야 한다.

아래와 같은 코드가 있다고 하자. 클라이언트는 직사각형의 사이즈를 변경하기 위해 `resize()` 메시지를 아래와 같이 전달한다고 하자.



``` java title="Client.java"
class Client {
    void resize(Ractangle ractangle, int width, int height) {
        ractangle.setWidth(width);
        ractangle.setHeight(height);
    }
}
```

직사각형은 가로와 세로의 길이가 다른 사각형이다. 가로와 세로의 길이를 설정하여 초기화한다. 

``` java title="Ractangle.java"
class Ractangle {
    protected int width;
    protected int height;
    
    public Ractangle(int width, int height) {
        this.width = width;
        this.height = height;
    }

    void setWidth(int width){
        this.width = width;
    }

    void setHeight(int height){
        this.height = height;
    }

    int getWidth() {
        return width;
    }

    int getHeight() {
        return height;
    }
}
```

이제 정사각형을 구현해보자. 수학에서는 정사각형은 직사각형 개념 안에 포함되어 있으니 상속을 해도 문제가 없어 보일 것이다.

``` java title="Square.java"

class Square extends Ractangle {
    protected int size;
    
    public Square(int size) {
        // width와 height를 동일하게 설정
        super(size, size);
    }

    @Override
    void setWidth(int width){
        width = width;
        height = height;
    }

    @Override
    void setHeight(int height){
        width = width;
        height = height;
    }
}
```

클라이언트 입장에선 Ractangle을 사용하는 경우만 가정했기 때문에 테스트 코드를 다음과 같이 작성할 수 있다. 

``` java title="Client.java"
class Client {
    void testResize(Ractangle ractangle) {
        // given 
        int width = 500;
        int height = 700;
        
        // when
        ractangle.setWidth(width);
        ractangle.setHeight(height);

        // then
        assert ractangle.getWidth() == width && ractangle.getHeight() == height;
    }
}
```

하지만 문제는 Square가 Ractangle의 자식 클래스이기 때문에 발생한다. 자바는 컴파일 시점과 런타임 시점의 코드가 달라지기 때문에 Ractangle의 자식 클래스인 Square로 testResize를 테스트하게 된다면 테스트는 실패하게 된다.

``` java
jshell> Ractangle ractangle = new Ractangle(5,2);
ractangle ==> Ractangle@58d25a40

jshell> testResize(ractangle); // 테스트 성공

jshell> Square square = new Square(5);
square ==> Square@1b9e1916

jshell> testResize(square); // 테스트 실패
|  Exception java.lang.AssertionError 
|        at testResize (#6:11)
|        at (#10:1)
```

왜 이러한 상황이 발생하는가? 그 결론은 **위와 같이 작성한 프로그램 세계의 정사각형은 직사각형 개념에 포함되지 않기 때문**이다.

즉 정사각형은 직사각형의 자식 클래스로서 기능이 호환되지 않는다고 본다.

## 결론

위의 세 예제를 통해 상속을 올바르게 작성 하지 못하는 근본적인 원인은 **클라이언트가 어떻게 사용할지 고민하기 보단, 프로그램의 구현 측면에서 개발자가 가정한 예제가 잘못되었을 때 문제가 발생하기 때문이다.** 

우리 프로그래머의 할 일은, **자신 혼자 고민하는 코드를 작성하기 보다는, 클라이언트가 어떻게 사용해주길 원하는 그러한 프로그램을 고려해야 한다.**

이러한 결론은 조금 성급하다고 생각된다. 아마 전달될 때 오해의 소지가 있어 보인다.

그러나, 방향은 맞다고 생각한다. 내가 작성한 코드를 상대방이 사용해주지 않으면 무용지물이기 때문이다.

그러한 부분은 필자가 조금 더 성장할 때 좀 더 표현을 가공하길 바라고 여기까지 왔음을 스스로에게 칭찬하면 마치도록 하자.
