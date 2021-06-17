## 1. The IoC Container

### IoC 컨테이너와 빈

첫 챕터에서는 IoC(Inversion of Control)의 원리를 가진 스프링 프레임워크의 구현을 다룬다.  
IoC는 DI(Dependency Injection)라고도 알려져 있다.  

스프링 컨테이너는 빈을 생성할 때 필요한 의존성을 모두 주입한다.  
이 과정은 빈이 자신의 생성이나 의존성에 대해 컨트롤하지 않는 과정이므로 기능적으로 제어의 역전(IoC)인 것이다.  

`org.springframework.beans` 와 `org.springframework.context` 패키지는 스프링 IoC 컨테이너의 기본 패키지이다.  
BeanFactory 인터페이스는 모든 타입의 객체들을 수용하는 설정을 제공한다.  
ApplicationContext는 BeanFactory의 서브 타입으로, 다음을 포함한다.  

- Spring AOP 기능의 손쉬운 통합
- Message Resource 핸들링
- 이벤트 발행
- WebApplicationContext와 같이, 웹 애플리케이션에서 사용되는 특정 응용 계층 컨텍스트

스프링에서는, 우리의 애플리케이션에 기반을 두고, IoC 컨테이너에 의해 관리되는 객체를 빈이라 부른다.  
빈은 스프링 IoC 컨테이너에 의해 생성되고, 응집되고, 관리되는 객체이다.  
반면에, 빈은 애플리케이션의 수많은 객체들 중 하나일 뿐이다.  
여러 빈들과, 그에 따르는 의존성들은 컨테이너를 통해 설정 메타데이터에 반영된다.  

### 컨테이너

`org.springframework.context.ApplicationContext` 인터페이스는 스프링 IoC 컨테이너를 의미하고, 이는 빈들을 생성하고, 설정하고, 조립하는 역할을 한다.  
컨테이너는 설정 메타데이터를 읽어서 어떤 객체들이 생성되고, 구성되고, 만들어져야 하는지를 알게 된다.  
설정 메타데이터는 XML, 자바 애노테이션, 혹은 자바 코드로 표현된다.  

우리의 애플리케이션은 컨텍스트가 생성되고 초기화된 이후에, 설정 메타데이터와 병합하게 된다.  
그러면 우리는 완전히 구성되고 실행가능한 시스템(애플리케이션)을 얻게 된다.  

![](./images/IoC_Container.png)  

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4d588756-ec5a-4293-a62e-da3dbec8bc87/IoC_Container.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4d588756-ec5a-4293-a62e-da3dbec8bc87/IoC_Container.png)

위의 설정 메타데이터는 우리 애플리케이션 개발자들에게 어떻게 스프링 컨테이너가 객체들을 생성하고, 구성할 수 있는지를 명시한다.  

설정 메타데이터는 XML 형식만 있는 것이 아니다.  
스프링 IoC 컨테이너는 설정 문서 형식과는 완전히 분리되어 있다.  
애노테이션 기반 설정은 스프링 2.5부터 시작되었고, `@Configuration`, `@Bean`, `@Import`, `@DependsOn` 과 같은 자바 기반 설정은 스프링 3.0부터 도입되었다.  

ApplicationContext는 여러 빈들과 그 의존성을 관리하고 유지하기 위한 팩토리 인터페이스이다.  
`T getBean(String name, Class<T> requiredType)` 메서드를 사용해서 빈 인스턴스를 조회할 수 있다.  
ApplicationContext 인터페이스는 빈을 조회하기 위한 여러 다른 메서드들도 제공하지만, 사실 애플리케이션에서 사용할 일은 없다.  
실제로 애플리케이션에서 getBean() 메서드를 사용할 일이 없고, 스프링 API에 의존적일 필요도 없다.  
예를 들어, 스프링 통합 웹 프레임워크는 다양한 웹 프레임워크 컴포넌트를 위한 DI를 제공하기 때문에, 굳이 직접적인 API를 호출할 필요는 없다.  

### 빈

빈의 정의는 `BeanDefinition` 객체로 표현되는데, 다음과 같은 메타데이터들을 포함하고 있다.  

- 패키지 기반 클래스 이름
- 빈의 행동 구성 요소(스코프, 라이프사이클 콜백 등)
- 해당 빈이 자신의 역할을 수행하기 위한 다른 참조 빈들(의존성)
- 새로운 객체를 생성할 때의 설정값(ex. 풀 사이즈, 커넥션 풀의 커넥션 수 등)

### 의존성

엔터프라이즈급 애플리케이션은 단지 하나의 객체로만 구성되지 않는다.  
간단한 애플리케이션조차 끝단 유저가 보는 것들을 표현하기 위해 여러 객체가 협업하고 있다.  

의존성 주입(DI)은 생성자, 팩토리 메서드, 혹은 인스턴스 생성 후의 설정된 속성을 통해서 종속성을 정의하는 프로세스다.  
컨테이너는 빈을 생성할 때 이런 의존성들을 주입한다.  
이 과정은 기본적으로 빈 자신의 역전(제어의 역전)으로, 빈의 의존성을 제어하는 과정이다.  

DI 방식을 사용하면 코드도 훨씬 깔끔하고, 이런 디커플링은 객체가 의존성을 필요로 할 때 더 효과적이다.  
객체는 의존성을 찾지도, 해당 객체가 어디에 있는지도 알지 못한다.  
결과적으로, 클래스는 테스트하기 쉬워지고, 특히 의존성이 인터페이스이거나 추상클래스인 경우 단위 테스트에서 테스트 대역을 사용할 수 있다.  

Constructor-based DI is accomplished by the container invoking a constructor with a number of arguments, each representing a dependency. Calling a static factory method with specific arguments to construct the bean is nearly equivalent

Constructor argument resolution matching occurs by using the argument’s type

Setter-based DI is accomplished by the container calling setter methods on your beans after invoking a no-argument constructor or a no-argument static factory method to instantiate your bean.

- Constructor-based or setter-based DI?

The Spring team generally advocates constructor injection, as it lets you implement application components as immutable objects and ensures that required dependencies are not null.

Furthermore, constructor-injected components are always returned to the client (calling) code in a fully initialized state. As a side note, a large number of constructor arguments is a bad code smell, implying that the class likely has too many responsibilities and should be refactored to better address proper separation of concerns.

- Circular dependencies

If you use predominantly constructor injection, it is possible to create an unresolvable circular dependency scenario.

---

you can define bean properties and constructor arguments as references to other managed beans (collaborators) or as values defined inline.
Spring’s XML-based configuration metadata supports sub-element types within its `<property/>` and `<constructor-arg/>` elements for this purpose.

The Spring container can autowire relationships between collaborating beans. You can let Spring resolve collaborators (other beans) automatically for your bean by inspecting the contents of the `ApplicationContext`. Autowiring has the following advantages:

- Autowiring can significantly reduce the need to specify properties or constructor arguments. (Other mechanisms such as a bean template [discussed elsewhere in this chapter](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-child-bean-definitions) are also valuable in this regard.)
- Autowiring can update a configuration as your objects evolve. For example, if you need to add a dependency to a class, that dependency can be satisfied automatically without you needing to modify the configuration. Thus autowiring can be especially useful during development, without negating the option of switching to explicit wiring when the code base becomes more stable.

---

### 빈 스코프

빈의 정의를 보고 빈을 생성할 때, 의존성과 설정값들 뿐만 아니라 객체의 생애 주기(scope)도 결정할 수 있다.  
이런 접근은 굉장히 강력하고 유연한 기능인데, 자바 클래스 레벨에서 객체의 생애 주기를 지정하지 않고, 그저 스코프를 객체에 알맞게 선택만 하면 되기 때문이다.  

스프링 프레임워크는 다음 6가지 스코프를 지원한다.  
그 중 4개는 웹 기반 `ApplicationContext` 에서 사용 가능한 스코프이다.  
물론 커스텀한 스코프도 만들 수 있다.  

- singleton
    - (기본값) IoC 컨테이너마다 딱 하나씩만 존재하는 빈
- prototype
    - 여러 개의 인스턴스가 생성될 수 있는 스코프
- request
    - 단건 HTTP 요청에 따른 생애주기를 가짐
    - 웹 기반 `ApplicationContext` 에서 사용 가능
- session
    - HTTP Session 단위로 생애주기를 가짐
    - 웹 기반 `ApplicationContext` 에서 사용 가능
- application
    - `ServletContext` 단위로 생애주기를 가짐
    - 웹 기반 `ApplicationContext` 에서 사용 가능
- websocket
    - `WebSocket` 단위로 생애주기를 가짐
    - 웹 기반 `ApplicationContext` 에서 사용 가능

싱글턴 빈은 단 하나만 만들어지고 공유되며, 해당 빈을 id로 참조하고 있는 모든 요청이 스프링 컨테이너에 의해 반환되는 1개의 특정 인스턴스를 받게 된다.  
즉, 스프링 컨테이너는 싱글턴으로 정의된 빈은 단 1개만 생성한다.  
생성된 하나의 인스턴스는 싱글턴 빈으로 캐싱되었다가 모든 하위 요청과 참조 위치에 반환된다.  

싱글턴이 아닌 프로토타입 스코프는 매 요청 시 새로운 빈 인스턴스를 만들게 된다.  
다른 스코프들과 다르게, 스프링은 프로토타입 빈의 생애주기를 완전히 관리하지 않는다.  
컨테이너는 프로토타입 빈을 원하는 요청을 받을 때, 생성에만 관여한 이후로는 책임을 클라이언트에게 넘긴다.  
그에 따라 클라이언트는 프로토타입 빈을 정리하고, 할당된 높은 비용의 리소스들을 해소해야만 한다.  
즉, 스프링 컨테이너의 역할은 프로토타입 빈의 `new` 연산자에만 관여하는 것이다.  

프로토타입 빈을 싱글턴 빈에 주입하게 되면 프로토타입 빈이 단 한번만 생성되게 되니 주의해야 한다.  

request, session, application, websocket 스코프는 웹 기반 `ApplicationContext` 에서만 사용 가능하다.  
이 스코프들은 싱글턴, 프로토타입 스코프와 다르게 약간의 초기 설정이 필요하다.  
스프링 MVC를 사용하고 있다면, 즉 `DispatcherServlet` 에 의해 다뤄지는 요청이라면, 특별한 세팅은 필요 없다.  
그 외 Servlet 2.5 컨테이너를 사용하고 있다면, RequestContextListener 등록을 해줘야 하고, Servlet 3.0 이상의 버전을 사용하고 있다면, WebApplicationInitializer 인터페이스를 사용하면 된다.  

request 스코프는 HTTP 요청 단위로 빈을 생성하고, session 스코프는 HTTP Session 단위로 빈을 생성한다.  
application 스코프는 ServletContext 단위로 빈이 생겨 싱글턴과 유사하지만, ApplicationContext 단위가 아니라 ServletContext 단위라는 점이 다르고, ServletContext의 속성으로 드러난다는 것이 다르다.  

커스텀한 스코프는 `org.springframework.beans.factory.config.Scope` 인터페이스를 사용하면 구현할 수 있다.
