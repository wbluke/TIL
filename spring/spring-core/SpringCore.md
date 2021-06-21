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

생성자 기반 DI는 필요한 의존성을 나타내는 생성자의 인자들을 기반으로 구성된다.  
스태틱 팩토리 메서드도 마찬가지다.  

setter 기반 DI는 기본 생성자나 기본 스태틱 팩토리 메서드를 사용해 빈 인스턴스를 만든 후, setter 메서드로 의존성을 주입하는 방식이다.  

스프링 팀에서는 일반적으로 setter DI보다 생성자 DI를 선호하는데, 이는 컴포넌트를 불변으로 만들어주고 의존성이 null이 아님을 보장해주기 때문이다.  
게다가, 생성자 DI 컴포넌트는 항상 클라이언트에게 완전히 초기화된 상태의 인스턴스를 보장한다.  
부작용이라면, 많은 수의 생성자 파라미터는 좋지 않은 코드 스멜을 야기한다는 것이다.  

또한 생성자 DI를 사용한다면, 순환 참조의 발생 가능성도 존재하게 된다.  

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

### 컨테이너 확장 포인트

`BeanPostProcessor` (빈 후처리기) 인터페이스는 초기화 로직, 의존성 로직 등을 재정의할 수 있도록 하는 콜백 메서드들이다.  
만약 스프링 컨테이너가 빈을 초기화한 후에 특별한 커스텀 로직을 수행하도록 추가하고 싶다면, 필요한 만큼 BeanPostProcessor 구현을 추가하면 된다.  

여러 개의 BeanPostProcessor 인스턴스를 구성할 때에는, `order` 속성으로 인스턴스 간 순서를 지정할 수 있다.  
이는 `Ordered` 인터페이스로 지정 가능하며, BeanPostProcessor를 사용할 때 항상 같이 고려하는 것이 좋다.  

BeanPostProcessor는 2개의 콜백 메서드를 가지는데, 각각 빈이 생성되기 전과 후를 제어할 수 있는 메서드들이다.  
빈 후처리기는 이 메서드를 통해 빈의 생성에 대한 어떤 처리도 진행할 수 있다.  
몇몇 스프링 AOP에서는 빈 후처리기를 사용해 프록시 래핑 로직을 수행하기도 한다.  

다음은 `BeanFactoryPostProcessor` 이다.  
BeanPostProcessor와 비슷하지만, 가장 큰 차이점은, BeanFactoryPostProcessor는 빈 설정 메타데이터에서 동작한다는 것이다.  
이는 스프링 IoC 컨테이너가 BeanFactoryPostProcessor가 설정 메타데이터를 읽고 변경할 수 있도록 허용한다는 뜻이다.  

BeanPostProcessor와 마찬가지로, 여러 개의 BeanFactoryPostProcessor를 사용하는 경우 `Ordered` 인터페이스로 순서를 같이 고려하는 것이 좋다.  

빈 팩토리 후처리기는 ApplicationContext에 선언되어 있는 경우 컨테이너에 정의된 설정 메타데이터를 변경하기 위해 자동으로 동작한다.  
스프링에는 이미 정의된 여러 빈 팩토리 후처리기가 존재하며, 커스텀한 빈 팩토리 후처리기를 추가할 수도 있다.  

`FactoryBean` 인터페이스는 빈을 생성할 수 있는 인터페이스이다.  
만약 어떤 빈의 초기화 로직이 복잡해서, XML 설정 등으로 구성하기가 어려운 경우 커스텀한 FactoryBean을 구성해서 추가할 수 있다.  
FactoryBean은 다음 3가지 메서드를 가진다.  

- `T getObject()`
    - 해당 팩토리가 생성한 빈 인스턴스를 반환한다.
    - 팩토리가 싱글턴을 반환하는지 프로토타입을 반환하는지에 따라 인스턴스를 공유할 수 있다.
- `boolean isSingleton()`
    - 싱글턴 빈을 반환하는지의 여부
- `ClasS<?> getObjectType()`
    - 생성하는 빈의 타입

### 어노테이션 기반 컨테이너 설정

> '어노테이션 기반 설정이 XML 설정 보다 나을까?' 라는 질문에 대한 답은 '개발자에게 달렸다'이다.  
어노테이션은 짧고 간결한 설정을 통해 선언만으로 많은 맥락을 제공하는 반면, XML은 본연의 소스 코드를 건드리지 않고 설정을 할 수 있다는 특징을 가지고 있다.  
어떤 개발자들은 소스 코드와 설정을 하나로 묶는 반면 또 다른 이들은 어노테이션 기반 클래스들이 더이상 POJO가 아니며, 설정들이 군집화되지 않고 제어하기 힘들다고 주장한다.  
스프링은 두 가지 스타일 모두 수용하고 있고, 동시 사용도 허용하고 있으니 상황에 맞게 사용하면 된다.

태그를 사용하는 XML 설정의 대안으로 바이트코드 수준에서 컴포넌트들을 연결하는 어노테이션 기반 설정이 있다.  
XML로 빈 와이어링을 설정하는 대신, 어노테이션을 통해 설정값들을 컴포넌트 클래스로 옮기는 방식이다.  

`@Required` 어노테이션은 setter 메서드에 쓰인다.  

```java
@Required
public void setMovieFinder(MovieFinder movieFinder) {
    this.movieFinder = movieFinder;
}
```

이는 초기 빈 설정 시 빈 정의나 오토와이어링을 통해 필요한 빈 프로퍼티가 있음을 알려주는 어노테이션이다.  

`@Autowired` 는 다음과 같이 사용한다.  

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

생성자 뿐만 아니라 setter 메서드, 필드에도 사용할 수 있다.  

오토와이어링 시에는 여러 빈 후보들이 있을 수 있기 때문에, 어떤 빈을 선택할지에 대한 제어가 필요하다.  
이를 구성하는 어노테이션 중 하나가 바로 `@Primary` 이다.  
`@Primary` 는 여러 후보 빈 중 특정 빈을 우선적으로 선택할 수 있도록 알려주는 어노테이션이다.  

```java
@Configuration
public class MovieConfiguration {

    @Bean
    @Primary
    public MovieCatalog firstMovieCatalog() { ... }

    @Bean
    public MovieCatalog secondMovieCatalog() { ... }

    // ...
}
```

좀 더 세밀한 조정을 원한다면, `@Qualifier` 어노테이션을 사용해볼 수도 있다.  
`@Primary` 가 하나의 우선 후보가 존재할 때 사용하면 좋은 반면, `@Qualifier` 는 전달한 파라미터 기반으로 어떤 빈을 선택할지 정할 수 있는 어노테이션이다.  

```java
public class MovieRecommender {

    @Autowired
    @Qualifier("main")
    private MovieCatalog movieCatalog;

    // ...
}
```

커스텀한 Qualifier 어노테이션을 만들 수도 있다.  

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre {

    String value();
}

@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface MovieQualifier {

    String genre();

    Format format(); // Enum
}
```
