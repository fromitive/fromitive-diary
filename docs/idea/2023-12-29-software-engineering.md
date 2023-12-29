---
title: "상속이 어떻게 중복 코드를 만들고 결합도를 높힐까?"
description: "상속을 하게되면 함수 내부의 구체적인 사항을 알아야 한다."
time: 2023-12-29 17:47:57
tags:
  - java
  - software-engineering
---

## 상속은 결합도를 높힌다?

코드를 재사용하지 않기 위해 `상속`을 사용한다고 한다. 상속을 하게 되면 부모 클래스의 메소드를 자식이 그대로 물려 받을 수 있기 때문이다. 이러한 상속은 결합도를 높히는 특징이 있다고 한다. 부모 클래스의 메소드를 알아야 하기 때문이라고 한다. 재사용하지 않기 위해 상속을 사용하면 코드의 중복이 나타날 수 있다고 한다.

아래의 코드를 한 번 보자 해당 예제는 하나의 휴대폰에 통화 요금을 계산하는 역할이 부여된 경우이다.

``` java title="Phone.java"
class Phone {
    // 통화 요금 계산
    public Money calculateFee() {
        Money result = Money.ZERO;
        // 요금을 계산하는 코드 e.g) 통화요금 * (총 통화 초 / 정책 초) 
        calls.stream().forEach(call -> result.plus(amount.times(call.getDuration().getSeconds() / seconds.getSeconds())))
        return result;
    }
}
```

여기서 아래와 같은 하나의 요구사항을 추가한다.

```
심야요금제를 운영하게 되었고, 10시 이후의 통화 요금이 할인된다. 
```

이를 해결하기 위해 Phone 클래스를 상속받아 `NightlyDiscountPhone`을 만들게 된다.

## 코드를 중복하면 어떤 일이 발생하게 되는가

`LATE_NIGHT_HOUR` 보다 시간이 넘으면 nightlyAmount 요금을 더하고, 그렇지 않은 경우는 `Phone`과 똑같이 일반 요금을 더하는 것이다.

코드의 재사용을 줄이기 위해 상속을 하였지만, **오히려 오버라이딩을 하면서 코드가 중복되었다.** 즉 이렇게 오버라이딩 된 코드는 `Phone`에서 `calculateFee`의 로직이 변경되었을 때 변경된 내용을 적용하지 못할 확률이 증가하게 된다. 

코드의 재사용은 로직이 변경될 때, 제대로 반영이 되지 않을 위험이 존재하게 된다.

``` java title="NightlyDiscountPhone.java"
class NightlyDiscountPhone extends Phone {
    @Override
    public Money calculateFee() {
        Money result = Money.ZERO;
        // 요금을 계산하는 코드 e.g) 통화요금 * (총 통화 초 / 정책 초) 
        calls.stream().forEach(call -> {
                if(call.getform.getHour() >= LATE_NIGHT_HOUR) {
                    // 심야 요금 계산
                    result = result.plus(nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds()))
                } else {
                    // 일반 요금 계산
                    result = result.plus(amount.times(call.getDuration().getSeconds() / seconds.getSeconds()))
                }
            })
        return result;
    }
}
```

## 코드 재사용을 위한 상속의 문제점 

코드의 중복을 피하기 위해 부모 클래스인 `Phone`의 메소드를 사용하면 아래와 같은 코드가 만들어진다.
**심야 요금을 계산하기 위해서 개발자는 `Phone`의 내부 구조를 아는 가정하에 코드를 짜게 된다.** 즉, 일반 요금이 심야 할인보다 높다는 가정을 새우고 코드를 작성하게 되어 부모 클래스인 Phone과의 결합도를 증가하도록 설계하게 된다.

이렇게 한 개의 객체를 만들 때 해당 객체의 사전 지식이 많을 경우 결합도가 높아지는 코드를 작성하는 확률이 올라가게 된다. 상속을 하게 되면 개발자가 세운 가정들을 정확히 알아야 하고 코드 수정을 어렵게 만들게 된다.

``` java title="NightlyDiscountPhone.java"
class NightlyDiscountPhone extends Phone {
    // **심야 요금 계산법** 
    // 심야 요금 2원
    // 일반 요금 5원
    // 10시에 10초 통화 = 20원 
    // 50원 - 30원 = 20
    @Override

    public Money calculateFee() {
        // 일반 요금 계산
        Money result= super.calculateFee();
        
        // 심야 요금 계산
        Money nightlyFee = Money.ZERO;
        calls.stream()
             .filter(call -> call.getform.getHour() >= LATE_NIGHT_HOUR)
            .forEach(call -> nightlyFee = getAmount().minus(nightlyAmount).times(
                call.getDuration().getSeconds() / getSeconds().getSeconds()
            ))

        // 심야 이후의 요금 만큼 뺀다. 
        return result.minus(nightlyFee);
    }
}
```

## 어떻게 해결해야 할까?

### 코드의 중복을 확인하고 함수로 만들자

`Phone`과 `NightlyDiscountPhone`의 중복된 코드는 `요금을 계산하는 것`이다. `요금을 계산 하는 것`을 추상화 시켜보면 공통대상이 보이기 시작한다. 어떤가? `calculateFee()` 부분의 로직이 같아졌다.

``` java title="Phone.java"
class Phone {
    // 통화 요금 계산
    public Money calculateFee() {
        Money result = Money.ZERO;
        calls.stream().forEach(call -> result.plus(calcluateCallFee))
        return result;
    }

    private Money calcluateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds())
    }
}
```

``` java title="Phone.java"
class NightlyDiscountPhone extends Phone {
    @Override
    public Money calculateFee() {
        Money result = Money.ZERO;
        calls.stream().forEach(call -> result.plus(calculateCallFee()))
        return result;
    }

    @Override 
    private Money calculateCallFee(Call call) {
        if(call.getform.getHour() >= LATE_NIGHT_HOUR) {
            // 심야 요금 계산
            return nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds())
        }
        // 일반 요금 계산
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds())

    }
}
```

### 상속을 할 때도 추상화에 의존하자

이제 `Phone` 을 상속한 `NightlyDiscountPhone` 관계를 해제하고 그보다 더 높은 객체인 `AbstractPhone`을 만들어 추상화하면 아래처럼 `Phone` 과 `NightlyDiscoundPhone`은 `AbstractPhone`을 상속받게 된다.  

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

이렇게 공통된 부분을 찾고 부모 클래스로 올리는 방식은 설계가 추상화에 의존하게 되고, 추상화 된 부분은 다른 객체가 사용할 때 구현을 신경쓰지 않고 활용할 수 있는 장점이 있다. 따라서 재사용성이 높아지는 코드가 만들어진다고 생각한다.

그리고, 이러한 `위로 올리기` 전략은 아래와 같은 이점이 있다고 한다.

```
설계를 변경하기 전에는 자식 클래스인 NightlyDiscountPhone이 부모 클래스인 Phone의 구현에 강하게 결합돼 있었기 때문에 Phone의 구현을 변경하더라도 NightlyDiscountPhone도 함께 영향을 받았었다는 점을 기억하라. 변경 후에 자식 클래스인 Phone과 NightlyDiscountPhone은 부모 클래스인 AbstractPhone의 구체적인 구현에 의존하지 않는다. 오직 추상화에만 의존한다. 정확히는 부모 클래스에서 정의한 calculateCallFee에만 의존한다. calculateCallFee의 시그니처가 변경되지 않는 한 부모 클래스의 내부 구현이 변경되더라도 자식 클래스는 영향을 받지 않는다. 이 설계는 낮은 결합도를 유지하고 있다.  
```

즉, `calcuateFee()`의 내부 로직이 변경되더라도, `NightlyDiscountPhone`과 `Phone`은 영향을 받지 않는다. 이는 부모 클래스가 추상화 되어있기 때문에 가능한 것이다. 따라서 **재사용할 수 있는 코드를 작성할 수 있는 핵심은 바로 추상화**임을 깨닫게 되었다. 

상속을 하더라도, 함수의 내부 구현을 알지 않음을 가정하면서 작성하는 습관을 기르자.