---
title: "가급적 상속 보단 합성을 이용해 책임을 전가하자"
description: "책임을 전가하여 높은 응집도와 낮은 결합도를 챙기는 방법"
time: 2023-12-30 16:38:46
tags:
  - java
  - software-engineering
---

## 지난 시간

어제의 글애선 상속을 이용하여 요금을 계산하는 부분을 추상화 하는 방법을 아래와 같이 사용할 수 있게 되었다. 오늘은, 상속이 아닌 합성을 통해 `요금을 계산`하는 책임을 한 곳으로 모아 응집도 높은 코드를 짜볼 예정이다.

``` java title="AbstractPhone.java" 
abstract class AbstractPhone {
    public Money calculateFee() {
        Money result = Money.ZERO;
        // 요금을 계산하는 코드 e.g) 통화요금 * (총 통화 초 / 정책 초) 
        calls.stream().forEach(call -> result.plus(calculateCallFee()))
        return result;
    }
    // 구현을 위해 접근 제어자를 protected로 변경
    abstract protected Money calculateCallFee(Call call); 
}

class Phone extends AbstractPhone { ... };
class NightlyDiscoundPhone extends AbstractPhone { ... };
```

## 합성을 쓰면 안좋은 점

책에서는 합성을 쓰는 좋은점을 많이 나열했다. 그리고 그걸 익히기 위해 예제 코드로도 잘 익히고 그랬지만, 합성의 단점을 생각해 봤다.

합성을 하게 되면 재사용 가능한 코드가 탄생하겠지만, 코드의 복잡도가 올라갈 수 있다. 하나의 책임을 다른 객체로 분리를 하고, 응집도를 높히기 때문이라고 생각한다.

또한, 런타임 의존성과, 컴파일 의존성이 다르기 때문에 `정적 분석`이 어려울 수 있다. 즉, 코드의 동작을 예측하기가 힘들어 진다고 생각한다. 보통 합성을 할 때 어떤 메시지를 보내야 할지 생각하므로 구체적인 구현 보다는, 인터페이스를 합성하곤 한다. 또한 그 방법이 결합도를 낮추고 책임이 집중되기에 응집도가 높아지는 특성이 있다.

그러나, 실행 할 때와, 표면적으로 소스코드는 다른 동작을 수행하기에 굳이 안좋은 점을 말하자면 실시간으로 디버깅하기 어렵다고 생각이 된다.

트레이드 오프라고 말하지만, 어느 수준에서 읽기 좋은 코드를 만들어야 할 지는 좋은 개발자의 센스에 달려있다고 생각된다.

## 합성을 이용하여 책임을 분리하기

위의 예시 처럼 요금에 대한 계산이 점점 복잡해진다고 하면 별도의 클래스를 하나 만들어 합성하는 방법을 사용할 수 있다. 아래는 `AbstractPhone`을 합성하는 방식으로 변경한 `Phone`이다. 요금 정책을 `RatePolicy`로 받았으며 구현되지 않았기에 요금에 관련된 정책을 임의로 만들기 편해졌다. 

``` java title="Phone.java" hl_lines="3 10" 
abstract class Phone {

    private RatePolicy ratePolicy;

    public Phone(RatePolicy ratePolicy) {
        this.ratePolicy = ratePolicy;
    }

    public Money calculateFee() {
        return ratePolicy.cacluateFee(this);
    } 
}

interface RatePolicy {
    Money calculateFee(Phone phone);
}
```

## 얼마나 편해졌는데?

calculateFee가 구현이 되어있지 않으면 아래와 같은 유연한 정책을 넣을 수 있다. 그러므로 그루(Guru)들이 상속 보다는 합성으로 이점을 가지라고 조언을 해주는 것을 어느정도 이해하게 되었다.

```
1. 요금 정책에 세금 정책을 부여하도록 넣기

2. 심야 요금 / 일반 요금 등 다양한 기본 요금 정책 구현

3. 요금 정책끼리 조합한 요금 정책

4. 조합한 요금 정책들 중 제일 할인율이 많은 정책으로 유연하게 변경하는 등
```

`calcuateFee`가 어떻게 개발이 되었든 위의 내용 말고도 여러 정책들을 조합하여 사용 가능하게 되었다.