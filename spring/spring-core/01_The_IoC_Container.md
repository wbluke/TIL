## IoC 컨테이너와 빈

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

## 컨테이너

`org.springframework.context.ApplicationContext` 인터페이스는 스프링 IoC 컨테이너를 의미하고, 이는 빈들을 생성하고, 설정하고, 조립하는 역할을 한다.  
컨테이너는 설정 메타데이터를 읽어서 어떤 객체들이 생성되고, 구성되고, 만들어져야 하는지를 알게 된다.  
설정 메타데이터는 XML, 자바 애노테이션, 혹은 자바 코드로 표현된다.  

우리의 애플리케이션은 컨텍스트가 생성되고 초기화된 이후에, 설정 메타데이터와 병합하게 된다.  
그러면 우리는 완전히 구성되고 실행가능한 시스템(애플리케이션)을 얻게 된다.  

![](./images/IoC_Container.png)  

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4d588756-ec5a-4293-a62e-da3dbec8bc87/IoC_Container.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4d588756-ec5a-4293-a62e-da3dbec8bc87/IoC_Container.png)

### 설정 메타데이터

위의 설정 메타데이터는 우리 애플리케이션 개발자들에게 어떻게 스프링 컨테이너가 객체들을 생성하고, 구성할 수 있는지를 명시한다.  

설정 메타데이터는 XML 형식만 있는 것이 아니다.  
스프링 IoC 컨테이너는 설정 문서 형식과는 완전히 분리되어 있다.  
애노테이션 기반 설정은 스프링 2.5부터 시작되었고, `@Configuration`, `@Bean`, `@Import`, `@DependsOn` 과 같은 자바 기반 설정은 스프링 3.0부터 도입되었다.  

ApplicationContext는 여러 빈들과 그 의존성을 관리하고 유지하기 위한 팩토리 인터페이스이다.  
`T getBean(String name, Class<T> requiredType)` 메서드를 사용해서 빈 인스턴스를 조회할 수 있다.  
ApplicationContext 인터페이스는 빈을 조회하기 위한 여러 다른 메서드들도 제공하지만, 사실 애플리케이션에서 사용할 일은 없다.  
실제로 애플리케이션에서 getBean() 메서드를 사용할 일이 없고, 스프링 API에 의존적일 필요도 없다.  
예를 들어, 스프링 통합 웹 프레임워크는 다양한 웹 프레임워크 컴포넌트를 위한 DI를 제공하기 때문에, 굳이 직접적인 API를 호출할 필요는 없다.  

## 빈

빈의 정의는 `BeanDefinition` 객체로 표현되는데, 다음과 같은 메타데이터들을 포함하고 있다.  

- 패키지 기반 클래스 이름
- 빈의 행동 구성 요소(스코프, 라이프사이클 콜백 등)
- 해당 빈이 자신의 역할을 수행하기 위한 다른 참조 빈들(의존성)
- 새로운 객체를 생성할 때의 설정값(ex. 풀 사이즈, 커넥션 풀의 커넥션 수 등)

## 의존성

엔터프라이즈급 애플리케이션은 단지 하나의 객체로만 구성되지 않는다.  
간단한 애플리케이션조차 끝단 유저가 보는 것들을 표현하기 위해 여러 객체가 협업하고 있다.  

### 의존성 주입

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

## 빈 스코프

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

### 싱글턴 스코프

싱글턴 빈은 단 하나만 만들어지고 공유되며, 해당 빈을 id로 참조하고 있는 모든 요청이 스프링 컨테이너에 의해 반환되는 1개의 특정 인스턴스를 받게 된다.  
즉, 스프링 컨테이너는 싱글턴으로 정의된 빈은 단 1개만 생성한다.  
생성된 하나의 인스턴스는 싱글턴 빈으로 캐싱되었다가 모든 하위 요청과 참조 위치에 반환된다.  

### 프로토타입 스코프

싱글턴이 아닌 프로토타입 스코프는 매 요청 시 새로운 빈 인스턴스를 만들게 된다.  
다른 스코프들과 다르게, 스프링은 프로토타입 빈의 생애주기를 완전히 관리하지 않는다.  
컨테이너는 프로토타입 빈을 원하는 요청을 받을 때, 생성에만 관여한 이후로는 책임을 클라이언트에게 넘긴다.  
그에 따라 클라이언트는 프로토타입 빈을 정리하고, 할당된 높은 비용의 리소스들을 해소해야만 한다.  
즉, 스프링 컨테이너의 역할은 프로토타입 빈의 `new` 연산자에만 관여하는 것이다.  

프로토타입 빈을 싱글턴 빈에 주입하게 되면 프로토타입 빈이 단 한번만 생성되게 되니 주의해야 한다.  

### Request, Session, Application, WebSocket 스코프

request, session, application, websocket 스코프는 웹 기반 `ApplicationContext` 에서만 사용 가능하다.  
이 스코프들은 싱글턴, 프로토타입 스코프와 다르게 약간의 초기 설정이 필요하다.  
스프링 MVC를 사용하고 있다면, 즉 `DispatcherServlet` 에 의해 다뤄지는 요청이라면, 특별한 세팅은 필요 없다.  
그 외 Servlet 2.5 컨테이너를 사용하고 있다면, RequestContextListener 등록을 해줘야 하고, Servlet 3.0 이상의 버전을 사용하고 있다면, WebApplicationInitializer 인터페이스를 사용하면 된다.  

request 스코프는 HTTP 요청 단위로 빈을 생성하고, session 스코프는 HTTP Session 단위로 빈을 생성한다.  
application 스코프는 ServletContext 단위로 빈이 생겨 싱글턴과 유사하지만, ApplicationContext 단위가 아니라 ServletContext 단위라는 점이 다르고, ServletContext의 속성으로 드러난다는 것이 다르다.  

커스텀한 스코프는 `org.springframework.beans.factory.config.Scope` 인터페이스를 사용하면 구현할 수 있다.  

## 컨테이너 확장 포인트

### BeanPostProcessor를 사용한 빈 커스터마이징

`BeanPostProcessor` (빈 후처리기) 인터페이스는 초기화 로직, 의존성 로직 등을 재정의할 수 있도록 하는 콜백 메서드들이다.  
만약 스프링 컨테이너가 빈을 초기화한 후에 특별한 커스텀 로직을 수행하도록 추가하고 싶다면, 필요한 만큼 BeanPostProcessor 구현을 추가하면 된다.  

여러 개의 BeanPostProcessor 인스턴스를 구성할 때에는, `order` 속성으로 인스턴스 간 순서를 지정할 수 있다.  
이는 `Ordered` 인터페이스로 지정 가능하며, BeanPostProcessor를 사용할 때 항상 같이 고려하는 것이 좋다.  

BeanPostProcessor는 2개의 콜백 메서드를 가지는데, 각각 빈이 생성되기 전과 후를 제어할 수 있는 메서드들이다.  
빈 후처리기는 이 메서드를 통해 빈의 생성에 대한 어떤 처리도 진행할 수 있다.  
몇몇 스프링 AOP에서는 빈 후처리기를 사용해 프록시 래핑 로직을 수행하기도 한다.  

### BeanFactoryPostProcessor를 사용한 설정 메타데이터 커스터마이징

다음은 `BeanFactoryPostProcessor` 이다.  
BeanPostProcessor와 비슷하지만, 가장 큰 차이점은, BeanFactoryPostProcessor는 빈 설정 메타데이터에서 동작한다는 것이다.  
이는 스프링 IoC 컨테이너가 BeanFactoryPostProcessor가 설정 메타데이터를 읽고 변경할 수 있도록 허용한다는 뜻이다.  

BeanPostProcessor와 마찬가지로, 여러 개의 BeanFactoryPostProcessor를 사용하는 경우 `Ordered` 인터페이스로 순서를 같이 고려하는 것이 좋다.  

빈 팩토리 후처리기는 ApplicationContext에 선언되어 있는 경우 컨테이너에 정의된 설정 메타데이터를 변경하기 위해 자동으로 동작한다.  
스프링에는 이미 정의된 여러 빈 팩토리 후처리기가 존재하며, 커스텀한 빈 팩토리 후처리기를 추가할 수도 있다.  

### FactoryBean을 사용한 초기화 로직 커스터마이징

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

## 어노테이션 기반 컨테이너 설정

> '어노테이션 기반 설정이 XML 설정 보다 나을까?' 라는 질문에 대한 답은 '개발자에게 달렸다'이다.  
어노테이션은 짧고 간결한 설정을 통해 선언만으로 많은 맥락을 제공하는 반면, XML은 본연의 소스 코드를 건드리지 않고 설정을 할 수 있다는 특징을 가지고 있다.  
어떤 개발자들은 소스 코드와 설정을 하나로 묶는 반면 또 다른 이들은 어노테이션 기반 클래스들이 더이상 POJO가 아니며, 설정들이 군집화되지 않고 제어하기 힘들다고 주장한다.  
스프링은 두 가지 스타일 모두 수용하고 있고, 동시 사용도 허용하고 있으니 상황에 맞게 사용하면 된다.

태그를 사용하는 XML 설정의 대안으로 바이트코드 수준에서 컴포넌트들을 연결하는 어노테이션 기반 설정이 있다.  
XML로 빈 와이어링을 설정하는 대신, 어노테이션을 통해 설정값들을 컴포넌트 클래스로 옮기는 방식이다.  

### @Required

`@Required` 어노테이션은 setter 메서드에 쓰인다.  

```java
@Required
public void setMovieFinder(MovieFinder movieFinder) {
    this.movieFinder = movieFinder;
}
```

이는 초기 빈 설정 시 빈 정의나 오토와이어링을 통해 필요한 빈 프로퍼티가 있음을 알려주는 어노테이션이다.  
Spring 5.1부터는 Deprecated 되었다.  

### @Autowired

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

### @Primary를 사용한 오토와이어링

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

### @Qualifier를 사용한 오토와이어링

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

### Qualifier와 제네릭 사용

Qualifer는 제네릭 타입도 감지한다.  
`Store<String>` 과 `Store<Integer>` 를 각각 구현한 Store가 있다고 가정할 경우, 아래와 같이 사용 가능하다.  

```java
@Configuration
public class MyConfiguration {

    @Bean
    public StringStore stringStore() {
        return new StringStore();
    }

    @Bean
    public IntegerStore integerStore() {
        return new IntegerStore();
    }
}
// -----------------------------------------
@Autowired
private Store<String> s1; // <String> qualifier

@Autowired
private Store<Integer> s2; // <Integer> qualifier
```

### @Value

`@Value` 는 외부 프로퍼티를 주입할 수 있는 어노테이션이다.  

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("${catalog.name}") String catalog) {
        this.catalog = catalog;
    }
}
// -----------------------------------------
// application.properties
catalog.name = MovieCatalog
```

### @PostConstruct와 @PreDestroy

`@PostConstruct` 는 빈 생성 전 초기화 로직을 추가할 수 있는 어노테이션이고, `@PreDestroy` 는 빈 소멸 시에 필요한 로직을 추가할 수 있는 어노테이션이다.  

```java
public class CachingMovieLister {

    @PostConstruct
    public void populateMovieCache() {
        // 초기화 로직
    }

    @PreDestroy
    public void clearMovieCache() {
        // 클리어 로직
    }
}
```

## Classpath 스캔과 컴포넌트 관리

지금까지 위 예제들에서는 빈에 대한 정의를 XML 기반으로 진행했지만, 이번에는 클래스패스를 스캔하여 후보 컴포넌트를 찾는 방식을 알아보려고 한다.  

### @Component와 표준 어노테이션

스프링은 `@Component` 어노테이션과 `@Repository` , `@Service` , `@Controller` 등의 `@Component` 어노테이션의 특수 케이스인 어노테이션들을 제공한다.  
`@Component` 를 사용하면 스프링에 의해 관리되는 컴포넌트, 혹은 에스펙트(aspect)를 적용하도록 해당 클래스를 사용할 수 있고, `@Repository` , `@Service` , `@Controller` 는 각 레이어에 맞게 사용할 수 있다.  

### 메타 어노테이션과 합성 어노테이션

스프링에서 제공되는 여러 어노테이션들은 코드에서 메타 어노테이션으로 활용된다.  
예를 들어, `@Service` 어노테이션은 `@Component` 어노테이션을 가진 메타 어노테이션이다.  
또한 메타 어노테이션은 여러 어노테이션이 합쳐져서 만들어지기도 한다.  
`@RestController` 는 `@Controller` 와 `@ResponseBody` 가 합쳐진 메타 어노테이션이다.  

### 클래스 자동 탐지 및 빈 정의 등록

스프링은 이런 클래스들을 자동으로 찾아서 ApplicationContext를 통해 BeanDefinition 을 만들어 등록할 수 있다.  
빈들을 찾고 등록하기 위해서는 `@Configuration` 클래스에 `@ComponentScan` 어노테이션을 추가해야 한다.  

빈 스캔 시에 원하는 타입을 정해서 필터도 적용할 수 있다.  
특정 클래스를 상속받는 어노테이션이라던가, Aspectj 표현식, 정규식, 커스텀한 필터 등으로 필터링하여 스캔할 수 있다.  

```java
@Configuration
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    ...
}
```

### 컴포넌트를 통한 빈 메타데이터 정의

스프링 컴포넌트는 또한 컨테이너에게 빈 정의 메타데이터를 구성해서 넘겨줄 수 있다.  
이를 `@Configuration` 클래스에서 `@Bean` 어노테이션을 사용해 특정 빈을 정의하여 컨테이너에 넘겨줄 수 있다.  

```java
@Component
public class FactoryMethodComponent {

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

}
```

일반 스프링 컴포넌트의 `@Bean` 메서드와 `@Configuration` 내의 `@Bean` 메서드는 차이가 있다.  
차이점은 `@Component` 클래스가 메서드나 필드 호출을 가로채기 위해서 CGLIB를 사용하지 않는다는 것이다.  

### 오토디텍팅된 컴포넌트 네이밍

컴포넌트가 디텍팅되면, 빈의 이름은 BeanNameGenerator를 통해 생성된다.  
스프링의 기본 제공 어노테이션(`@Component` , `@Repository` , `@Service` , `@Controller` )들은 이에 따라 해당 빈 정의에 이름을 제공한다.  

### 오토디텍팅된 컴포넌트에 스코프 제공

스프링이 관리하는 컴포넌트는 기본적으로 `싱글턴` 스코프를 갖는다.  
그러나 때로 다른 스코프를 지정해야 할 때가 있는데, 이때는 `@Scope` 어노테이션을 사용하면 된다.  

```java
@Scope("prototype")
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

## JSR 330 표준 어노테이션

스프링 3.0을 사용한다면, 스프링은 JSR-330 표준 어노테이션의 지원을 제공한다.  
이 어노테이션들은 스프링 어노테이션과 마찬가지로 스캐닝되며, 사용하려면 관련 jar 파일을 클래스패스에 추가해야 한다.  

`@Autowired` 대신 `@Inject` 를 사용해볼 수도 있다.  
`@Autowired` 와 마찬가지로, `@Inject` 도 필드 레벨, 메서드 레벨, 생성자 레벨에 적용할 수 있다.  
게다가 주입 포인트에 Provider를 선언하면 주입받은 빈에 대한 지연 로딩을 구현할 수도 있다.  

```java
import javax.inject.Inject;

public class SimpleMovieLister {

    private Provider<MovieFinder> movieFinder;

    @Inject
    public void setMovieFinder(Provider<MovieFinder> movieFinder) {
        this.movieFinder = movieFinder;
    }

}
```

`@Named` 와 `@ManagedBean` 은 `@Component` 대신 사용해볼 수도 있다.  

```java
@Named("movieListener")  // @ManagedBean("movieListener")
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

}
```

스프링의 어노테이션을 대체할 수 있는 나머지 JSR-330 어노테이션에 대한 소개와 제약 사항은 다음과 같다.  

- `@Autowired`
    - `@Inject` 로 대체 가능
    - `@Inject` 에는 required 필드가 없어서, 필요하다면 Java8의 Optional을 같이 활용해야 한다.
- `@Component`
    - `@Named` , `@ManagedBean` 으로 대체 가능
    - 구성 가능한 모델을 제공하지 않고 이름으로 식별하는 방법만 제공
- `@Scope("singleton")`
    - `@Singleton`
    - JSR-330의 기본 스코프는 스프링의 prototype과 비슷하게 동작하지만, 스프링 컨테이너 내에서는 싱글턴으로 동작하도록 선언되었다.
- `@Qualifier`
    - `@Qualifier` , `@Named`
- `@Value` , `@Required` , `@Lazy`
    - 대체할 수 있는 사항이 없다.
- ObjectFactory
    - Provider

## Java 기반 컨테이너 설정

이번 섹션에서는 자바 코드에서 스프링 컨테이너 설정을 어떻게 해야하는지를 알아본다.  

### @Bean과 @Configuration

스프링의 새로운 자바 구성 지원의 중심 요소는 `@Configuration` 가 적용된 클래스와 `@Bean` 이 적용된 메서드이다.  
`@Bean` 은 메서드에서 스프링 컨테이너에서 관리할 새로운 객체를 생성하고, 구성하고, 초기화할 때 사용한다.  
`@Bean` 이 달린 메서드는 어느 `@Component` 클래스에서도 사용 가능하지만, 보통은 `@Configuration` 클래스에서 사용된다.  

`@Configuration` 선언의 가장 주된 목적은 빈 정의의 원천임을 나타내는 것이다.  
`@Configuration` 클래스 내에서 다른 `@Bean` 메서드를 호출하여 빈 간 종속성을 정의할 수 있다.  

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

> `@Configuration` 이 선언되지 않은 클래스에서 `@Bean` 메서드가 선언되면 `lite` 한 모드에서 처리되는 것으로 인식된다.  
예를 들어 서비스 컴포넌트는 적용 가능한 각 컴포넌트 클래스에 대한 라이트 `@Bean` 선언을 통해 컨테이너에 관리 뷰를 노출할 수 있다.  
일반적인 `@Configuration` 내 `@Bean` 선언과 달리, 라이트 `@Bean` 메서드는 빈 간 종속성을 선언할 수 없다.  
그래서 이런 `@Bean` 메서드는 다른 `@Bean` 메서드를 호출해서는 안 된다.  
일반적인 시나리오에서는 `@Bean` 메서드는 `@Configuration` 클래스 내에서 선언되어 "full" 모드로 사용되고, 컨테이너의 라이프사이클 관리 하에 각 메서드 참조가 동작하도록 하는 것이 좋다.

### AnnotationConfigApplicationContext를 사용한 스프링 컨테이너 초기화

다음은 스프링 3.0에서 소개된 `AnnotationConfigApplicationContext` 이다.  
이 다용도의 ApplicationContext 구현체는 `@Configuration` 클래스를 받아들일 수 있을 뿐만 아니라 `@Component` 클래스나 JSR-330 어노테이션 클래스도 받아들일 수 있다.  

`@Configuration` 클래스가 입력으로 제공되면 해당 클래스 자체가 빈 정의로 등록되고 클래스 내에 선언된 모든 `@Bean` 메서드도 빈 정의로 등록된다.  
`@Component` 및 JSR-330 클래스가 제공되면 빈 정의로 등록되고, 필요한 경우 `@Autowired` 나 `@Inject` 와 같은 DI 메타 데이터를 사용하는 것으로 가정한다.  

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

또는 다음과 같이 register()로 등록할 수도 있다.  

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();

    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

컴포넌트 스캔을 원한다면 다음과 같이 패키지 기반 범위를 지정할 수 있다.  

```java
@Configuration
@ComponentScan(basePackages = "com.acme") 
public class AppConfig  {
    ...
}
```

위 예제는 "com.acme" 패키지 하위의 모든 `@Component` 클래스를 찾아서 컨테이너에 빈 정의로 등록한다.  
`AnnotationConfigApplicationContext` 에서도 scan() 메서드로 위 스캐닝을 진행할 수 있다.  

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```

### @Bean 어노테이션 사용

`@Bean` 어노테이션으로 메서드 레벨에서 메서드가 반환하는 타입으로 빈 정의를 등록할 수 있는데, 이때 기본적으로 메서드의 이름이 빈의 이름이 된다.  

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    }
}
```

만약 아래와 같이 구현체를 인터페이스 타입으로 반환한다면, 빈 등록은 가능하지만 막상 해당 인터페이스에 여러 구현체가 존재할 경우 어떤 구현체를 빈으로 가질지 문제가 생길 수 있으니 가능하면 구체적인 구현체 타입으로 빈을 등록하는 것이 좋다.  

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```

`@Bean` 으로 정의된 클래스는 라이프사이클 콜백을 지원하고, JSR-250의 `@PostConstruct` 와 `@PreDestroy` 를 사용할 수 있다.  
또한 다음과 같이 초기화 메서드와 소멸 메서드를 지정할 수도 있다.  

```java
public class BeanOne {

    public void init() {
        // initialization logic
    }
}

public class BeanTwo {

    public void cleanup() {
        // destruction logic
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public BeanOne beanOne() {
        return new BeanOne();
    }

    @Bean(destroyMethod = "cleanup")
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

또 필요하다면 name 프로퍼티로 이름을 지정하거나, alias를 지정할 수 있고, `@Description` 을 통해 빈에 대한 설명도 작성할 수 있다.  

### @Configuration 어노테이션 사용

`@Configuration` 메서드는 클래스 레벨에서 사용되며, `@Bean` 메서드를 통해 빈을 선언하는 역할을 한다.  

```java
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        return new BeanOne(beanTwo());
    }

    @Bean
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

> 위와 같이 메서드를 통한 빈 간 종속성 설정은 `@Configuration` 클래스 내의 `@Bean` 메서드끼리만 가능하다.  
일반 `@Component` 클래스 내에서는 빈 간 종속성을 선언할 수 없다.

```java
@Configuration
public class AppConfig {

    @Bean
    public ClientService clientService1() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientService clientService2() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientDao clientDao() {
        return new ClientDaoImpl();
    }
}
```

만약 위와 같이 인터페이스 타입(ClientDao)을 서로 다른 두 메서드에서 사용하고 있다면 (싱글턴 스코프에서) 두 개의 인스턴스가 생성되어 문제가 될 것 같이 보인다.  
실제로는 `@Configuration` 클래스는 시작 시 CGLIB를 사용하여 서브클래싱되고, 하위 클래스에서 자식 메서드는 부모 메서드를 호출하고 새 인스턴스를 만들기 전에 캐싱된 빈이 있는지 확인하기 때문에 문제가 발생하지 않는다.  

### 자바 기반 설정 구성

`@Import` 어노테이션은 여러 모듈화된 설정 파일들 간에 사용하는데, 다른 설정 파일을 불러올 때 사용한다.  

```java
@Configuration
public class ConfigA {

    @Bean
    public A a() {
        return new A();
    }
}

@Configuration
@Import(ConfigA.class)
public class ConfigB {

    @Bean
    public B b() {
        return new B();
    }
}
```

위 예제에서는 ConfigB 파일만 컨텍스트에 로딩해도 A, B 빈들을 모두 사용할 수 있게 된다.  
이는 컨테이너 초기화 로직을 훨씬 간단하게 만들 수 있는 방법이다.  

사실 위 예제는 너무 간단하고, 좀 더 실용적인 시나리오를 살펴보자.  
많은 경우 빈들은 서로 의존성을 가지며, 다른 설정 파일에 해당 의존성이 존재하는 경우도 있다.  
XML 기반이라면 사실 문제될 것이 없는데, 컴파일러가 개입하지 않고, `ref` 속성으로 필요한 빈을 선언해줄 수 있기 때문이다.  
하지만 다행히도, `@Bean` 메서드는 필요한 빈 의존성을 파라미터로 받을 수 있기 때문에 문제를 해결할 수 있다.  

```java
@Configuration
public class ServiceConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}
```

시스템 환경에 따라 설정 파일들을 조건적으로 선택할 수도 있다.  
`@Profile` 어노테이션은 더 유연한 구조의 `@Conditional` 어노테이션을 통해 구현되고 있고, 이는 `org.springframework.context.annotation.Condition` 클래스를 통해 동작한다.  
Condition 인터페이스의 구현부를 보면 다음과 같이 matches() 메서드로 프로파일 조건을 체크하고 있는 것을 볼 수 있다.  

```java
@Override
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    // @Profile 어노테이션 속성 읽어오기
    MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
    if (attrs != null) {
        for (Object value : attrs.get("value")) {
            if (context.getEnvironment().acceptsProfiles(((String[]) value))) {
                return true;
            }
        }
        return false;
    }
    return true;
}
```

## 환경 추상화

`Environment` 인터페이스는 컨테이너 내에서 통합된 추상 개념이고, profiles와 properties라는 두 가지 중요한 관점을 가지고 있다.  

profile은 주어진 프로파일에 따라 각각 등록되는 빈 정의들의 논리적 묶음이다.  
profile과 관련된 `Environment` 객체의 역할은 현재 어떤 프로파일이 동작하고 있는지, 그리고 어떤 프로파일이 기본 설정으로 동작하도록 되어있는지를 결정하는 것이다.  

properties는 거의 모든 애플리케이션에서 중요한 역할을 하고, 아마도 많은 소스(프로퍼티 파일, JVM 시스템 프로퍼티, 시스템 환경 변수, JNDI, 서블릿 컨텍스트 파라미터, 애드혹 프로퍼티 객체, 맵 객체 등)들의 기원이 되는 항목이다.  
properties와 관련된 `Environment` 객체의 역할은 프로퍼티를 구성하고 적용함으로써 사용자에게 편리한 서비스 인터페이스를 제공하는 것이다.  

### 빈 정의 프로파일

빈 정의 프로파일은 코어 컨테이너에서 서로 다른 환경에 서로 다른 빈이 등록될 수 있는 매커니즘을 제공한다.  
"환경"이라는 단어는 "서로 다른 사용자들을 위한 서로 다른 어떤 것"을 의미할 수 있는데, 이 기능은 다음과 같은 상황에 도움을 줄 수 있다.  

- 개발 환경에서는 인메모리 데이터소스를 사용하는 반면 QA나 프로덕션 환경에서는 JNDI 데이터소스를 사용하는 것
- 퍼포먼스 환경에 배포할 때만 모니터링 인프라스트럭처를 등록하는 것
- AB 테스트 시 각각 서로 다른 빈들을 구성하는 것

`DataSource` 의 예를 생각해보자.  
테스트 환경에서는 다음과 같이 환경을 구성할 수 있을 것이다.  

```java
@Bean
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(EmbeddedDatabaseType.HSQL)
        .addScript("my-schema.sql")
        .addScript("my-test-data.sql")
        .build();
}
```

다음으로 QA나 프로덕션 환경에서는 데이터소스가 애플리케이션 서버 내 JNDI 디렉토리에 등록되어있다고 가정해보자.  

```java
@Bean(destroyMethod="")
public DataSource dataSource() throws Exception {
    Context ctx = new InitialContext();
    return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
}
```

문제는 어떻게 현 환경에 맞게 두 개의 변수를 전환할 것인가이다.  
빈 정의 프로파일은 이런 문제에 대한 해결책을 제공하는 핵심 컨테이너 기능이다.  
위 사례를 일반화하자면 상황 A에서 빈 정의의 특정 프로파일을 등록하고, 상황 B에서는 다른 프로파일을 등록하고 싶다는 것으로 이야기할 수 있다.  

`@Profile` 어노테이션은 컴포넌트가 활성화된 1개 혹은 그 이상의 프로파일들에 맞게 빈을 등록할 수 있도록 한다.  

```java
@Configuration
@Profile("development")
public class StandaloneDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }
}

@Configuration
@Profile("production")
public class JndiDataConfig {

    @Bean(destroyMethod="")
    public DataSource dataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

프로파일 문자열은 `!` , `&` , `|` 와 같은 연산자와 함께 구성할 수 있다.  

또 다음과 같이 `@Profile` 어노테이션을 메타 어노테이션으로 활용해볼 수도 있다.  

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Profile("production")
public @interface Production {
}
```

`@Profile` 은 메서드 레벨에서도 등록할 수 있다.  

```java
@Configuration
public class AppConfig {

    @Bean("dataSource")
    @Profile("development") 
    public DataSource standaloneDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }

    @Bean("dataSource")
    @Profile("production") 
    public DataSource jndiDataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

이제 환경을 구성했으면, 스프링 애플리케이션을 프로파일에 맞게 실행시켜야 한다.  
프로파일 활성화는 여러가지 방식으로 수행할 수 있지만, 가장 간단한 방법은 ApplicationContext를 통해 사용할 수 있는 `Environment` API에 대해 프로그래밍 방식으로 수행하는 것이다.  

```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("development");
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();
```

추가적으로, 활성화할 프로파일을 `spring.profiles.active` 프로퍼티로 선언할 수 있다.  

```java
-Dspring.profiles.active="profile1,profile2"
```

마지막으로, 프로파일이 지정되지 않았을 때의 기본 프로파일은 다음과 같이 지정할 수 있다.  
프로파일이 지정된 상황이라면 기본 프로파일은 적용되지 않는다.  

```java
@Configuration
@Profile("default")
public class DefaultDataConfig {

    // ...
}
```

### PropertySource 추상화

스프링의 `Environment` 추상화는 프로퍼티 소스 계층 구조에 대한 검색을 제공한다.  

```java
ApplicationContext ctx = new GenericApplicationContext();
Environment env = ctx.getEnvironment();
boolean containsMyProperty = env.containsProperty("my-property");
System.out.println("Does my environment contain the 'my-property' property? " + containsMyProperty);
```

위 예제에서 현재 환경에 my-property 속성이 정의되어 있는지 스프링에 물어볼 수 있다.  
이를 답하기 위해 Environment 객체는 PropertySource 객체들의 집합에서 검색을 수행한다.  
`PropertySource` 는 키-값 쌍으로 이루어진 단순한 추상이며, 스프링의 `StandardEnvironment` 는 두 개의 PropertySource(JVM 환경 프로퍼티, 시스템 환경 프로퍼티)로 구성된다.  
결론적으로, `StandardEnvironment` 를 사용할 경우 `env.containsProperty("my-property")` 는 `my-property` 시스템 프로퍼티가 있거나 환경 변수가 있는 경우에 참을 반환할 것이다.  

### @PropertySource 사용

app.properties 파일에서 키-값 쌍을 정의한 경우 다음과 같이 해당 값을 가져올 수 있다.  

```java
@Configuration
@PropertySource("classpath:/com/myco/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

## ApplicationContext의 추가 기능

`org.springframework.beans.factory` 패키지는 프로그래밍 방식을 포함하여, 빈을 다루고 관리하는 기능을 제공한다.  
`org.springframework.context` 패키지는 BeanFactory 인터페이스를 상속하고, 프레임워크-지향 방식의 추가적인 기능들을 제공하는 다른 인터페이스들도 상속한 ApplicationContext 인터페이스를 추가했다.  
많은 사람들은 ApplicationContext를 프로그래밍 방식으로 생성하는 대신, ContextLoader와 같은 지원 클래스에 의존하여 자동으로 인스턴스화한다.  

보다 프레임워크-지향 방식으로 BeanFactory의 기능을 향상시키기 위해 컨텍스트 패키지는 다음과 같은 기능도 제공한다.  

- `MessageSource` 를 통한 i18n(internationalization)-스타일의 메시지 접근
- `ResourceLoader` 를 통한 URL과 파일과 같은 리소스 접근
- `ApplicationEventPublisher` 를 통한 이벤트 발행 ( `ApplicationListener` 를 구현한 빈들)
- `HierarchicalBeanFactory` 인터페이스를 통해 애플리케이션의 웹 계층과 같은 특정 계층에 각각 집중할 수 있도록 여러 계층 구조 컨텍스트를 로딩

### MessageSource를 통한 국제화

ApplicationContext는 MessageSource 인터페이스를 확장하고 있기 때문에 i18n 메세징 기능을 제공할 수 있다.  

- `String getMessage(String code, Object[] args, String default, Locale loc)`
    - 가장 기본이 되는 메서드로, MessageSource로부터 메시지를 조회한다.
    - 적절한 지역이 없으면 기본 메시지를 사용한다.
    - 라이브러리에 의해 제공되는 MessageFormat 기능을 사용해서 어떤 인자든 대체 값으로 넣어줄 수 있다.
- `String getMessage(String code, Object[] args, Locale loc)`
    - 위 메서드와 같으나 기본 메시지가 없어서, 적절한 지역 메시지가 없는 경우 NoSuchMessageException을 던진다.
- `String getMessage(MessageSourceResolvable resolvable, Locale locale)`
    - 위에서 사용된 모든 속성이 MessageSourceResolvable로 묶여서 사용된다.

ApplicationContext가 로딩될 때, 컨텍스트에 정의된 MessageSource를 자동으로 찾는다.  
그 빈은 반드시 이름이 `messageSource` 여야만 한다.  
빈을 찾으면, 위의 메서드를 통한 모든 호출은 메시지 소스로 위임된다.  
메시지 소스 빈이 없으면, ApplicationContext는 해당 빈의 이름을 가진 부모가 있는지 찾기 시작한다.  
만약 ApplicationContext가 아무 빈도 찾지 못한다면 비어있는 DeligatingMessageSource가  인스턴스화된다.  

```java
public static void main(String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("message", null, "Default", Locale.ENGLISH);
    System.out.println(message);
}
```

### 표준 이벤트와 커스텀 이벤트

ApplicationContext 의 이벤트 핸들링은 ApplicationEvent 클래스와 ApplicationListener 인터페이스에 의해 제공된다.  
만약 빈이 ApplicationListener 인터페이스를 구현하고 컨텍스트 내에 존재한다면, 항상 ApplicationEvent가 ApplicationContext에 발행될 때마다 빈에 알림이 전송된다.  

스프링에서 제공하는 표준 이벤트는 다음과 같다.  

- ContextRefreshedEvent
- ContextStartedEvent
- ContextStoppedEvent
- ContextClosedEvent
- RequestHandledEvent
- ServletRequestHandledEvent

커스텀 이벤트를 만들 수도 있다.  

```java
public class BlockedListEvent extends ApplicationEvent {

    private final String address;
    private final String content;

    public BlockedListEvent(Object source, String address, String content) {
        super(source);
        this.address = address;
        this.content = content;
    }

    // ...
}
```

ApplicationEvent를 발행하기 위해서는, ApplicationEventPublisher의 publishEvent() 메서드를 호출하면 된다.  
전형적으로 이는 ApplicationEventPublisherAware 를 구현한 클래스를 생성하고 빈으로 등록함으로써 수행된다.  

```java
public class EmailService implements ApplicationEventPublisherAware {

    private List<String> blockedList;
    private ApplicationEventPublisher publisher;

    public void setBlockedList(List<String> blockedList) {
        this.blockedList = blockedList;
    }

    public void setApplicationEventPublisher(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }

    public void sendEmail(String address, String content) {
        if (blockedList.contains(address)) {
            publisher.publishEvent(new BlockedListEvent(this, address, content));
            return;
        }
        // 이메일 전송
    }
}
```

스프링 컨테이너는 ApplicationEventPublisherAware를 구현한 EmailService 를 찾아서, setter를 통해 자기 자신을 주입한다.  
이 과정에서 ApplicationEventPublisher 인터페이스를 통해 애플리케이션 컨텍스트와 소통할 수 있게 된다.  

커스텀 ApplicationEvent를 받기 위해서는, ApplicationListener 를 구현한 클래스를 생성하고 스프링 빈으로 등록하면 된다.  

```java
public class BlockedListNotifier implements ApplicationListener<BlockedListEvent> {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    public void onApplicationEvent(BlockedListEvent event) {
        // notificationAddress를 통해 적절한 알림 발송
    }
}
```

ApplicationListener는 일반적으로 사용자가 지정한 타입으로 제네릭화된다.  
즉, onApplicationEvent() 메서드는 다운 캐스팅이 필요하지 않고 타입 안전하게 유지될 수 있다.  

원하는 만큼 이벤트 리스너를 등록할 수 있지만 기본적으로 이벤트 리스너는 이벤트를 동기식으로 수신한다.  
즉, 모든 리스너가 이벤트 처리를 완료할 때까지 publishEvent() 메서드가 차단된다.  
이런 동기 및 단일 스레드 방식의 장점 중 하나는 리스너가 이벤트를 수신할 때 트랜잭션 컨텍스트를 사용할 수 있는 경우 발행한 곳의 트랜잭션 컨텍스트 내에서 작동한다는 점이다.  
이벤트 수신에 대한 다른 전략이 필요하면 ApplicationEventMulticaster, SimpleApplicationEventMulticaster를 참고하면 된다.  

어노테이션 기반으로도 리스너를 등록할 수 있다.  
다음과 같은 경우 인터페이스를 구현할 필요도 없고, 메서드 이름도 자유로워진다.  

```java
public class BlockedListNotifier {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    @EventListener
    public void processBlockedListEvent(BlockedListEvent event) {
        // notificationAddress를 통해 적절한 알림 발송
    }
}
```

여러 개의 이벤트를 수신해야하거나 파라미터 없이 메서드를 정의하고 싶은 경우는 다음과 같이 지정할 수도 있다.  

```java
@EventListener({ContextStartedEvent.class, ContextRefreshedEvent.class})
public void handleContextStart() {
    // ...
}
```

이벤트 수신 후 또 다른 이벤트를 발행해야 한다면, 다음과 같이 반환값으로 이벤트를 줄 수도 있다.  

```java
@EventListener
public ListUpdateEvent handleBlockedListEvent(BlockedListEvent event) {
    // notificationAddress를 통해 적절한 알림 발송
    // 그리고 ListUpdateEvent 발행
}
```

특정 리스너가 비동기적으로 이벤트를 받아서 작업해야 한다면 다음과 같은 어노테이션을 사용할 수 있다.  

```java
@EventListener
@Async
public void processBlockedListEvent(BlockedListEvent event) {
    // BlockedListEvent가 별도의 스레드에서 수행된다.
}
```

비동기 이벤트는 다음과 같은 특징이 있으니 주의해야 한다.  

- 만약 비동기 이벤트 리스너가 예외를 던진다면, 이는 요청자에게까지 전파되지 않는다.
- 비동기 이벤트 리스너 메서드는 반환 값으로 다른 이벤트를 주어 새로운 이벤트를 발행할 수 없다.  
만약 다른 이벤트를 이어서 발행해야 한다면 수동으로 이벤트를 발행하기 위해 ApplicationEventPublisher를 주입해야 한다.

이벤트 리스너 간 순서를 지정할 수도 있다.  

```java
@EventListener
@Order(42)
public void processBlockedListEvent(BlockedListEvent event) {
    // notificationAddress를 통해 적절한 알림 발송
}
```

제네릭을 사용하여 이벤트 구조를 추가로 정의할 수도 있다.  
다음과 같이 `EventCreatedEvent<T>` 를 사용하면 생성되는 엔티티의 타입을 지정하여, Person에 대한 EntityCreatedEvent만 수신할 수 있다.  

```java
@EventListener
public void onPersonCreated(EntityCreatedEvent<Person> event) {
    // ...
}
```

### 낮은 수준의 리소스에 대한 편리한 접근

애플리케이션 컨텍스트는 Resource 객체를 로드할 수 있는 ResourceLoader이다.  
Resource는 JDK의 `java.net.URL` 클래스보다 더 본질적으로 많은 기능을 제공하는 클래스이며, 사실 Resource 구현체가 URL 클래스를 감싸고 있다.  
Resource는 클래스패스, 파일 시스템, 표준 URL 등 거의 모든 위치에 있는 낮은 수준의 리소스를 얻을 수 있다.  
만약 리소스의 위치 정보가 아무런 접두사 없이 사용되었다면, 애플리케이션 컨텍스트의 타입을 보고 적절한 리소스를 불러온다.  

당신은 빈 초기화 시점에 애플리케이션 컨텍스트를 ResourceLoader로 주입 받아서 사용하기 위해 ResourceLoaderAware를 사용할 수 있다.  
또한 정적 리소스에 접근하기 위해 Resource 타입 속성을 노출할 수도 있으며, 이는 다른 속성과 마찬가지로 주입될 것이다.  
이러한 리소스 속성을 간단한 문자열 경로로 지정하고 빈이 배포될 때 실제 리소스 객체로 자동 변환되는 것을 기대할 수 있다.  

## BeanFactory

### BeanFactory or ApplicationContext?

특별한 이유가 없는 한 BeanFactory 대신 ApplicationContext를 사용해야 한다.  
ApplicationContext는 BeanFactory의 모든 기능을 포함하기 때문에 일반적으로 빈 처리에 대한 완전한 제어가 필요하지 않은 이상 BeanFactory보다 권장된다.  

다음은 BeanFactory에서는 제공하지 않지만 ApplicationContext에서 제공하는 기능들이다.  

- 통합 라이프사이클 관리
- 자동 BeanPostProcessor 등록
- 자동 BeanFactoryPostProcessor 등록
- 편리한 MessageSource 접근
- 내재된 ApplicationEvent 발행 메커니즘
