---
title: "spring-framework 메모"
description: "spring-framework이 알아서 관리해 준다는데, 이게 기존 자바보다 좋은 점이 도대체 뭔데?"
time: 2023-12-20 23:10:28
tags:
  - java
  - spring-framework
---

## 스프링

웹 어플리케이션을 쉽게 만들어 주게 하는 프로젝트이다.  

## 특징 

### 낮은 결합도
사랑받는 이유는 Loose-coupling 특성이 존재하기 때문이다.
Java Enterprise Edition(Jakarta EE)가 웹 어플리케이션을 만들기 위해서는 사전 작업이 많이 필요했지만, 스프링은 기존에 만들어놓았던 인터페이스 객체를 그대로 사용할 수 있기 때문에 사용자들에게 많이 사랑받는다.

또한, 스프링에 의존하는 것이 아니라 자신의 비즈니스 로직에 집중하도록 도와주는 역할을 한다.

### Inversion-of-Control 컨테이너

스프링은 개발자가 만든 객체를 IoC 컨테이너를 통해 관리된다. 관리 단위를 Bean이라고 하는데 Bean으로 할 수 있는 행위는 다음과 같다.

객체에 접근하거나(getBeans)
스프링이 관리해야 할 객체들을 등록(@Component) 
객체를 생성하기 위해 필요한 객체를 자동으로 생성(@Autowired)하거나 
객체가 생성되거나 사라질때(@PreDestroy, @PostConstruct)처리 방식

## 프로젝트 종류

### Spring Webflux

비동기 웹 어플리케이션을 만들 때 사용한다.

### Spring MVC

일반 웹 어플리케이션 만들 때 사용한다.

### Spring Boot

유연한 Spring WebApplication을 만들때 사용한다. 제일 많이 사랑받는 프로젝트이다.

### Spring Security

보안에 관련된 모듈이다.

### Spring DataBase

Spring에서 DataBase 접근을 위해 필요한 프로젝트이다.

## 기본 사용법

### 1. IoC Container 생성

IoC Container 를 생성한다. 이 컨테이너가 Spring에서 사용할 Bean을 관리하며 종류는 BeanFactory, ApplicationContext 가 있다. ApplicationContext가 좀 더 기능이 많고, 편하게 관리할 수 있는 이점이 있다고 한다.

다음은 ApplicationContext를 생성하는 방법이다. AnnotationConfigApplicationContext로 ApplicationContext를 만들 수 있고 getBean() 등으로 Bean으로 등록된 객체에 접근이 가능하다.

``` java
try(var context = AnnotationConfigApplicationContext([ConfigurationAnnotion 클래스].class)){
	// context로 Bean객체 제어
	context.getBean(SomeBeanClass.class)
}
```

### 2. Configuration 등록

AnnotationConfigApplicationContext를 사용하기 위해서는 Configuration Annotation이 붙여진 객체를 써야 한다.
안써도 Spring측에서 자동으로 붙여줘서 관리 받을 수 있다.

``` java
@Configuration // 안 써도 그만!
class SomeConfiguration {

}
```

#### @ComponentScan

스프링이 컨텍스트를 사용하기 위해 검색할 수 있게 제공하는 Annotation이다. 아무것도 안써있으면 현재 패키지 기준으로 @Component Annotation이 붙여진 객체를 Bean(IoC컨테이너가 관리하는 객체)으로 만든다.

특정 패키지를 검색하기 위해서는 @ComponentScan(`패키지 명`)을 작성하면 사용 가능하다.

### 3. Bean 등록

스프링이 관리할 객체인 Bean을 만드는 방법은 @Bean 을 쓰거나, @Component를 사용하여 관리한다. 

@Bean
SomeBean getBean(){
	return someBean;
}

@Bean은 사용자가 관리하기 힘든 리소스를 설정할 때 사용한다고 한다. 

@Component
class SomeComponent {
}

보통 사용자들이 사용하기 위해 Component를 사용한다고 한다. 하지만, 좀 더 다른 팀원들이 이해하기 쉽도록 스테리오 타입 이 존재한다고 한다(스테리오 = 규칙이 있는 타입)

#### 스테리오 타입 종류

@Repository, @Service, @Controller

## Inversion-of-Control

### 의존성

하나의 객체 A를 동작시킬 때 필요한 객체들(B,C,D)이 있을 때, A는 BCD와 의존한다라고 말한다. 

#### 의존성 주입

A를 생성할 때, 외부의 객체 B,C,D를 인자로 받아 초기화 하는 과정(?)을 말한다 이렇게 하게 되면 아래의 이점이 있다.

느슨한 결합 - B,C,D를 직접 구현하지 않고 추상화 시킨다. B,C,D의 내부 구조를 쉽게 변경 가능하도록 만들어 준다.
<-> 강한 결합 - B,C,D를 직접 구현하여 의존하는 경우를 의미한다.

구현하지 않고 의존 관계만 정의하는 것. 이것이 느슨한 결합이라 생각한다.

### Auto-Wired

Bean에 의존하는 Bean이 있을때, 해당 Bean을 자동으로 생성하여 붙여(Auto-Wired)주는 작업이다. 즉, A를 초기화 할때 B,C,D를 자동으로 생성하여 붙여준다.

### 의존성 주입 종류

1. Constructor-based DI (Spring에서 기본적으로 처리됨)
2. Setter-based DI (@Autowired 필요)
3. field-based DI (@Autowired 필요)

### 생성 시점 변경
Spring의 Beans는 생성 시점 제어가 가능하다.

#### Eager

#### Lazy
@Lazy Annotiation을 붙이면 사용자가 필요할 때 초기화 된다고 한다.

### Bean Scope

Bean의 기본 생성 범위는 Single-ton 이라고 한다. 즉, 한 IoC Container 한 종류의 Bean이 초기화 된다.

이를 변경할 수 있는데 @Scope(value=ConfigurableBeanFactory.SCOPE_PROTOTYPE)으로 설정하면 사용할 때 마다. 같은 종류의 Bean을 생성한다.

클라이언트 별로 상태를 관리할 때 사용한다고 한다. 


### 소멸, 생성시 제어

#### PostConstruct
생성자가 호출된 이후 실행하는 코드를 지정한다. DB 초기 설정을 할때 사용한다고 한다.

#### PreDestruct
객체가 소멸될 때 실행하는 코드를 지정한다. 자원을 깔끔하게 지울 때 사용한다고 한다.

### 중복되는 Bean이 있을경우 우선권 설정

#### Quailfier

#### Priamry

### Jakarta CDI

Spring이 사용하는 Component나, AutoWire의 Interface이고, Spring Annotion이 Jakarta CDI의 구현체(implement)라고 한다.