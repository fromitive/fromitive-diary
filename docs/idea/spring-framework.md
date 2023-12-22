---
title: "spring-framework 메모"
description: "spring-framework이 알아서 관리해 준다는데, 이게 기존 자바보다 좋은 점이 도대체 뭔데?"
time: 2023-12-20 23:10:28
tags:
  - java
  - spring-framework
---

Create Object

Wireing Dependency

What is Wiring Dependency

GameRunner -> GameConsole

게임 러너는 게임 콘솔이 있어야 동작이 된다. --> Dependency


wiring??

Spring framework의 역할

객체의 상호간의 의존성을 관리해준다?? bean을 통해?
ConfigurationContext 안에서 Bean을 정의할 수 있고

ConfigurationContext 객체 생성
AnnotationConfigApplicationContext([Configuration Annotaion 클래스])
Bean은 Spring framework가 관리할 대상으로 인식한다고한다.

어떤 "관리"를 제공해주는데? 

- Auto-Wired
    @Bean
    public Person autoWiredPerson(String name, int age) {
        //관리하는 Bean내에 name과 age를 연결시켜 자동으로 집어 넣는 역할을 한다. 
        //이것을 Auto-Wired라고 한다.
        return new Person(name, age);
    }

- 관리하는 객체 조회
getBeanDefinition
getBeanDefinitionCount
- 중복하고 있는 객체를 관리
  : 스프링은 Type을 중복으로 쓰는 Bean이 있을때, getBeans를 사용하면 예외를 발생시킨다.
   : auto-wiring 을 쓸 때도 없는 Bean 이름을 쓰면 후보군을 찾게되는데 후보가 여러개일 때 예외를 발생시킨다.
  : 이를 해결하기 위해 Primary를 설정한다.
    - 설정하는 방법 @Bean에 @Primary Annotation 추가

   : 다른 해결방법으로 Quailfier를 사용하는 것이다.
    - Qualifier는 후보군을 지정하여 기본 값을 설정할 수 있는 장점이 있다.



Spring Context를 생성한 후 객체를 관리하는 방법
//1: Launch Spring context
//2: Configure the things that we want Spring to manage - @Configuration 을 통해

Bean 생성하는 방법


관리하고 있는 Bean 객체를 반환하는 방법
context.getBean("Bean 객체 이름"); 또는
context.getBean([Bean으로 설정한 클래스])


Spring Container
Spring 빈과 life-cycle을 관리하는 주체 
-> 만드는 법
@Configuration
@Bean
--> AnnotationConfigApplicationContext
-> 동일 단어
동일 언어
Spring Context
Application Context --> 관점 지향 프로그래밍, 웹 프로그래밍, 국제화 등 지원한다.
			(Spring Container 중 한개)
IOC(Inversion of Control: 제어의 역전) Container


java Bean vs Spring Bean
Spring이 객체를 자동을 생성하게 하는 방법?

POJO(Plain Old Java Object) - 일바 자바 객체?

스프링에서 자동으로 객체를 생성하고 관리하는 방법

1. 사용할 Bean - @Component Annotation 사용
2. Application Context 안에서 @ComponentScan 사용
  - @ComponentScan("패키지명")으로 지정도 가능!

Dependency Injection

만일, 관리하는 객체에 의존하는 다른 객체가 있을 경우 자동으로 Dependency Injection이 일어날 수 있다.

스프링에서 사용하는 Dependency Injection은 아래와 같다.

1. Constructor Injection (자동)
2. Setter based injection
3. field based injection

Constructure를 제외한 Setter based, Field based는 @AutoWired Annotation을 추가해준다.

Inversion of Control ->
main코드에서 프로그래머가 객체를 관리하는 것이 아닌
@Component, @Comfiguration 그리고, @Component 객체의 의존을 Auto-wire를 한 스프링이 객체의 관리를 담당하게 된다. 이를 Inversion of Control이라고 한다.


@Lazy Initialization
사용자가 @Lazy를 붙인 객체를 사용할 때, 초기화 되는 Bean을 생성한다.
- 마치 python interpreter와 같이 필요할 때 호출되는 느낌이다.
- 오류가 뒤늦게 발견되기도 한다.

@Eager Initializtion(default)

@Component와 @Bean Annotation에 기본으로 설정된 초기화 방식
 - 초기 오류를 즉시 잡아주어 대부분 권장한다.

Bean의 Scope
관리하는 Bean은 기본적으로 1개만 초기화된다.(single-ton)
 - Spring Singleton : IoC Container는 한 개의 Object Instance만 갖는다.

여러개의 객체의 상태값을 관리하기 위해서는 @Scope를 사용할 수 있다. -> 부를때 마다 메모리를 할당한다.

@Scope(value=ConfigurableBeanFactory.SCOPE_PROTOTYPE)

Prototype은 언제 사용하나? -> 상태를 관리할 때 사용

PostConstruct와 PreDestroy Annotation 

Jakarta CDI --> Context and Dependency Injection Specification

스테리오 스프링 어노테이션 
@Component: Generic Spring Annotation for Bean creation

@Service
@Controller
@Repository

---

@ComponentScan -> Component를 스캔한다.
@Compnent

@Primary
@Qualifier

@Autowired

@Lazy

@Postconstruct
@PreDestroy

@Scope

Jakarta Annotation
@Named = @Component
@Inject = @Autowired
