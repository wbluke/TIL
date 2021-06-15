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
