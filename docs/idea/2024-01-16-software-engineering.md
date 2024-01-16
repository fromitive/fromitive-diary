---
title: "결합도가 낮은 어플리케이션"
description: "확장할수록 바뀌는 부분과 그렇지 않는 부분을 나눠야 한다."
time: 2024-01-16 17:43:43
tags:
  - software-engineering
---

## 결합도가 낮고 확장성을 고려한 어플리케이션인데?

결합도가 낮더라도 **일관적이지 않으면, 유지보수 하기 힘든 프로그램이 된다는 것을 알게되었다.**

<<오브젝트>> 에서는 한개의 핸드폰에 통화 기록에 따라 요금 정책을 적용하여 계산하는 어플리케이션을 통해 이를 설명해준다.  

## 핸드폰 요금 정책 예제

핸드폰 1개가 있다고 가정하자, 아래의 핸드폰은, 핸드폰의 통화기록을 추가할 수 있고, 요금 정책에 따라, 툥화 요금을 반환한다.

``` java title="Phone.java"
class Phone {
    List<Call> calls = new ArrayList<>();
    RatePolicy ratepolicy;
    
    public Phone(RatePolicy ratepolicy) {
        this.ratepolicy = ratepolicy;
    }

    public Money calculateFee(){
        return ratepolicy.calculateFee(this);
    }

    public List<Call> getCalls() {
        return Collections.unmodifiableList(calls);
    }
}
```

Call은 `통화 시작 시간` 및 `통화 종료 시간`을 저장하는 from, to로부터 생성되어 진다. 이를 하나로 묶어 `DataTimeInterval`객체를 만들었다.

다양한 정책들을 만들기 위해 다양한 역할들이 추가되었다.

``` java title="Call.java"
public class Call {
    DateTimeInterval interval;

    public Call(LocalDateTime from, LocalDateTime to) {
        this.interval = DateTimeInterval.of(from, to);
    }

    public Duration getDuration() {
        return interval.duration();
    }

    public LocalDateTime getFrom() {
        return interval.getFrom();
    }

    public LocalDateTime getTo() {
        return interval.getTo();
    }

    public DateTimeInterval getInterval() {
        return interval;
    }

    public List<DateTimeInterval> splitByDay() {
        return interval.splitByDay();
    }
}

```

RatePolicy은 Phone요금을 계산해주는 역할을 수행한다.

``` java title="RatePolicy.java"
interface RatePolicy {
    Money calculateFee(Phone phone);
}
```

call의 시간대별로 요금 정책을 부가하기 위해 `BaseRatePolicy`를 만든다. 만든 내용은 아래와 같다.


``` java title="BasicRatePolicy.java"
public abstract class BasicRatePolicy implements RatePolicy {

    @Override
    public Money calculateFee(Phone phone) {
        return phone.getCalls().stream()
                .map(each -> calculateCallFee(each))
                .reduce(Money.ZERO, (first, second) -> first.plus(second));
    }

    protected abstract Money calculateCallFee(Call call);
}
```

Call안의 시간대를 이용하여 다양한 요금정책을 적용하기 위해선 `calculateCallFee`를 구현하기만 하면 된다.

### 고정 요금 정책 구현

통화 시작 및 끝의 시간차를 계산하여 기준 초당 기준 요금을 부과하는 방식이다. 

``` java title="FixedFeePolicy.java"
public class FixedFeePolicy extends BasicRatePolicy {
    private Money amount;
    private Duration seconds;

    public FixedFeePolicy(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}

```

### 시간대별 요금 정책 구현

각 시간대별로 별도의 기준 초와 요금을 설정하여 요금을 부과하는 방식이다. call안의 1개의 통화기록은 여러날에 거쳐서 통화할 수 있으므로 `splitByDay()`를 구현하였다.

`calculateIntervalFee`에서 `DateTimeInterval`리스트를 각 요금 테이블(starts, ends, durations, amounts)로 처리하는 모습을 보인다.

``` java title="TimeOfDayDiscountPolicy.java"

public class TimeOfDayDiscountPolicy extends BasicRatePolicy {
    // starts , ends , durations , amounts
    // 시작(시:분), 종료(시:분), 부가단위(초), 부가금액(원)

    private List<LocalTime> starts = new ArrayList<>();
    private List<LocalTime> ends = new ArrayList<>();
    private List<Duration> durations = new ArrayList<>();
    private List<Money> amounts = new ArrayList<>();

    @Override
    protected Money calculateCallFee(Call call) {
        return call.splitByDay().stream()
                .map(interval -> calculateIntervalFee(interval))
                .reduce(Money.ZERO, (first, second) -> first.plus(second));

    }

    private Money calculateIntervalFee(DateTimeInterval interval) {
        Money result = Money.ZERO;
        for (int loop = 0; loop < starts.size(); loop++) {
            result = result.plus(amounts.get(loop).times(
                    Duration.between(from(interval, starts.get(loop)), to(interval, ends.get(loop)))
                            .getSeconds() / durations.get(loop).getSeconds()));
        }
        return result;
    }

    private LocalTime to(DateTimeInterval interval, LocalTime to) {
        return interval.getTo().toLocalTime().isAfter(to) ? to : interval.getTo().toLocalTime();
    }

    private LocalTime from(DateTimeInterval interval, LocalTime from) {
        return interval.getFrom().toLocalTime().isBefore(from) ? from : interval.getFrom().toLocalTime();
    }

}

```

### 요일별 요금 정책 구현

요일별로 기준 초와 요금을 설정한 후, 요일마다 다른 요금을 부과하는 방식이다.

특이사항은 `시간대별 요금 정책 구현`과 같이 Call안에 있는 `DateTimeInterval`을 이용하여 요금을 부과하고 있으나, `DayOfWeekDiscountRule`을 여러개 만들어 한 Rule당 요금을 부과하고 있다.

이로 인해 `calcualteCallFee` 가 `시간대별 요금 정책 구현`내용과 곂친다.

``` java title="DayOfWeekDiscountPolicy.java"
public class DayOfWeekDiscountPolicy extends BasicRatePolicy {

    List<DayOfWeekDiscountRule> rules = new ArrayList<>();

    @Override
    protected Money calculateCallFee(Call call) {
        return call.splitByDay().stream()
                .map(interval -> calculateIntervalFee(interval))
                .reduce(Money.ZERO, (first, second) -> first.plus(second));
    }

    private Money calculateIntervalFee(DateTimeInterval interval) {
        return rules.stream()
                .map(rule -> rule.calculate(interval))
                .reduce(Money.ZERO, (first, second) -> first.plus(second));
    }

}
```

`DayOfWeekDiscountRune`은 DateTimeInterval을 처리하기 위해 `calculate` 메소드를 구현하고 있다. 각 요일을 저장하고 있고

정책이 적용되는 요일에 통화를 하고 있으면 그에따라, 요금이 부과하는 방식이다.

``` java title="DayOfWeekDiscountRule.java"
public class DayOfWeekDiscountRule {
    private List<DayOfWeek> dayOfWeeks = new ArrayList<>();
    private Duration duration = Duration.ZERO;
    private Money amount = Money.ZERO;

    public DayOfWeekDiscountRule(List<DayOfWeek> dayOfWeeks, Duration duration, Money amount) {
        this.dayOfWeeks = dayOfWeeks;
        this.duration = duration;
        this.amount = amount;
    }

    public Money calculate(DateTimeInterval interval) {
        if (dayOfWeeks.contains(interval.getFrom().getDayOfWeek())) {
            return amount.times(interval.duration().getSeconds() / duration.getSeconds());
        }
        return Money.ZERO;
    }

}

```

### 구간별 요금 정책 구현

마지막이다. 구간(Duration)별 요금 정책은 A초부터 B초까지 C원에 대한 여러 룰들을 합하여 최종적으로 부과한다.

`요일별 요금 정책 구현`과 같이 요금 룰을 `DurationDiscountRule`로 구현하여 여러 룰을 적용한다. 

``` java title="DurationDiscountPolicy.java"
public class DurationDiscountPolicy extends BasicRatePolicy {
    List<DurationDiscountRule> rules = new ArrayList<>();

    public DurationDiscountPolicy(List<DurationDiscountRule> rules) {
        this.rules = rules;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        return rules.stream().map(rule -> rule.calculate(call)).reduce(Money.ZERO,
                (first, second) -> first.plus(second));
    }

}
```

아래는 DurationDiscountRule이다. 이전 게시물에서 이야기하듯 잘못된 상속을 사용하고 있다. FixedFeePolicy에 기능이 추가가 된다면, 의도하지 않게 DurationDiscountRule에 기능이 추가될 가능성이 있는 코드다.

``` java title="DurationDiscountRule.java"
public class DurationDiscountRule extends FixedFeePolicy {
    private Duration from;
    private Duration to;

    public DurationDiscountRule(Money amount, Duration seconds, Duration from, Duration to) {
        super(amount, seconds);
        this.from = from;
        this.to = to;
    }

    public Money calculate(Call call) {
        if (call.getDuration().compareTo(to) > 0) {
            return Money.ZERO;
        }

        if (call.getDuration().compareTo(from) < 0) {
            return Money.ZERO;
        }

        Phone phone = new Phone();
        phone.call(new Call(call.getFrom().plus(from),
                        call.getDuration().compareTo(to) > 0 ? call.getFrom().plus(to) : call.getTo()));

        return super.calculateFee(phone);
    }
}
```

## 어디서부터 일관성이 사라진걸까?

BaseRatePolicy까지 구현했을 땐, 일관성이 있고 좋았다. 문제는 다음부터였다. 다양한 요금정책들이 각각 다른 요구사항이 있어 이를 구현하느라 많은 자율성이 부여가 되었다.

고정요금을 부과할 때만 해도 amount와 duration을 이용하여 적절히 구현할 수 있었지만  Call당 통화 기간을 기반으로 요금을 부과하면서 요일별, 구간별, 시간대별 등등 다양한 요구사항을 만족하기 위해 일관성이 없는 구조가 만들어졌다.

이를 바로잡지 않으면 소프트웨어는 점점 일관적이지 않는 구조로 바뀌게 될 것이며 유지보수가 어렵고, 이해하기가 어려운 코드가 만들어 질 가능성이 생기게 된다.

아래는 몇 가지 특징을 기반으로 공통사항이 있는지 비교한 표이다. 

| 구분             | 고정 요금 | 시간대별 요금 | 요일별 요금 | 구간별 요금 |
| ---------------- | --------- | ------------- | ----------- | ----------- |
| super            | -         | -             | -           | O           |
| DateTimeInterval | -         | O             | O           | -           |
| Rule             | -         | -             | O           | O           |
| amount, duration | O         | O             | O           | O           |

## 일관성 없는 구현

## 일관성을 부여하려면? 변하는 것을 찾아라
