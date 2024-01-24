---
title: "팩토리 메서드"
description: "객체 생성을 캡슐화 하는 3가지 방법"
time: 2024-01-24 20:44:06
tags:
  - software-engineering
  - design-pattern
---

```
팩토리의 3가지 방법

1. simple factory

2. factory method

3. abstract factory
```


팩토리 

인스턴스를 생성을 직접 하게 되면 코드의 유연성이 떨어지게 된다.

이때, 생성하는 역할을 전담하는 객체를 팩토리라고 한다.

일단, 팩토리를 생성하게 되면, 생성하는 곳이 한 곳이기 때문에 여러 생성자 메소드를 사용하는 곳을 대체한다면 일관성을 유지하기가 쉬워진다.

사용 방법

```java
Factory factory;

factory.create(type);

팩토리도 인터페이스 형태로 존재할 수 있음

interface Factory {
Pizza create(String type);
}

class NYPizzaFactory implements Factory {
	Pizza create(String type);
}

```
사용하는 클라이언트를 합쳐서 인터페이스화 해도 된다. 이를 팩토리 메소드라고 한다.
``` java
public abstract class PizzaStore {
	public Pizza orderPizza(String type){
		Pizza pizza =createPizza(type);	
	}
	abstract protected Pizza createPizza(String type);
}
```
이렇게 구성하면 서브 클래스를 구현하여 다양한 createPizza메소드를 구축할 수 있게 되고, 바뀌지 않는 orderPizza를 일관성 있게 유지할 수 있다. 즉, 바뀌는 부분을 createPizza로 캡슐화하였고, 바뀌지 않는 부분인 orderPizza를 일관적으로 유지한 것이다.

이를 팩토리 메소드로부터 생성되는 인스턴스는 해당 팩토리 객체의 서브클래스에서 결정된다. 또한, 생성하기 위한 인스턴스도, Pizza라는 일관된 인터페이스를 사용하므로 Pizza에서 제공하는 메시지만 보낼 수 있다.

팩토리를 자체를 추상화 할 수 있다. 이를 추상 팩토리라고 한다. 

추상 팩토리는 일관적으로 객체를 생성하도록 도와 줄 수 있다. 변화하는 부분은 세부 방법이 구현하는 것에 따라 달라지게 된다.
``` java
interface PizzaIngradientFactory {
	Sauce createSauce();
}
```
이 추상 팩터리를 NY나 시카고형태로 구현(implements)를 하면 전략 패턴처럼 추상화에 의존할 수 있게 된다.

``` java
class NYPizzaIngradientFactory implements PizzaIngradientFactory {
	@Override
	Sauce createSauce() {
		return new NYSauce();
	} 
}

```
