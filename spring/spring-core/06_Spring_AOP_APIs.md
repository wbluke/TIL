이번 장에서는 하위 레벨의 스프링 AOP API에 대해서 다뤄보고자 한다.  

## 스프링에서의 포인트컷 API

### 개념

스프링의 포인트컷 모델은 어드바이스 타입에 관계 없이 포인트컷 재사용을 가능하게 한다.  
당신은 같은 포인트컷으로 다른 어드바이스를 타켓팅할 수 있다.  

`org.springframework.aop.Pointcut` 인터페이스는 핵심 인터페이스로, 특정 클래스나 메서드에 어드바이스를 타겟팅하기 위해 사용된다.  

```java
public interface Pointcut {

    ClassFilter getClassFilter();

    MethodMatcher getMethodMatcher();
}
```

포인트컷 인터페이스를 두 부분으로 나눈 것은 클래스와 메서드 매칭 파트의 재사용과 조합 연산을 가능하게 한다.  

ClassFilter 인터페이스는 주어진 타깃 클래스 집합에서 포인트컷을 제한하는 데 사용된다.  
matches() 메서드가 항상 참을 반환한다면, 모든 타깃 클래스는 매칭된다.  

```java
public interface ClassFilter {

    boolean matches(Class clazz);
}
```

MethodMatcher는 일반적으로 더 중요하다.  

```java
public interface MethodMatcher {

    boolean matches(Method m, Class<?> targetClass);

    boolean isRuntime();

    boolean matches(Method m, Class<?> targetClass, Object... args);
}
```

matches(Method, Class) 메서드는 이 포인트컷이 대상 클래스의 주어진 메서드와 일치하는지 테스트하는 데에 사용된다.  
이 평가는 모든 메서드에 대해 테스트가 필요하지 않도록 AOP 프록시가 생성될 때 수행될 수 있다.  
두 개의 인자를 가진 matches() 메서드가 참을 반환하고 isRuntime() 메서드도 참을 반환하는 경우, 모든 메서드를 호출할 때마다 인자 세 개 짜리 matches() 메서드가 호출된다.  
이를 통해 포인트컷은 타깃 어드바이스가 시작되기 직전에 메서드 호출에 전달된 인자를 볼 수 있다.  

대부분의 MethodMatcher 구현은 정적이며, isRuntime() 메서드가 false를 반환한다.  
이 경우 인자 3개 짜리 matches()는 호출되지 않는다.  

## 스프링에서의 어드바이스 API

### 어드바이스 생명주기

각 어드바이스는 스프링 빈이다.  
어드바이스 인스턴스는 모든 어드바이스된 객체에서 공유하거나 각 객체에서 고유할 수 있다.  
이는 클래스별 어드바이스와 인스턴스별 어드바이스로 나뉜다.  

클래스별 어드바이스가 보통 흔하다.  
트랜잭션 어드바이저 같은 제네릭 어드바이스에 적절하다.  
이 어드바이스는 프록시된 객체의 상태의 의존적이지도 않고 새로운 상태를 추가하지도 않는다.  
단지 메서드와 인자에 따라서만 행동한다.  

인스턴스별 어드바이스는 믹스인 지원을 위한 인트로덕션에 적합하다.  
이 경우에는 어드바이스가 프록시 객체에 상태를 추가한다.  

당신은 동일한 AOP 프록시에서 공유 및 인스턴스별 어드바이스를 혼합해서 사용할 수 있다.  

### 스프링의 어드바이스 타입

스프링은 몇 가지의 어드바이스 타입을 제공하며 임의의 어드바이스 타입을 지원하도록 확정 가능하다.  

스프링에서의 가장 기본적인 어드바이스 타입은 어드바이스 전후 인터셉션이다.  

```java
public interface MethodInterceptor extends Interceptor {

    Object invoke(MethodInvocation invocation) throws Throwable;
}
```

> [역주]  
스프링 코드를 열어보면, MethodInterceptor 는 어드바이스이고, MethodInvocation은 조인 포인트이다.  
MethodInterceptor는 기본적으로 어라운드 어드바이스의 역할을 하고, MethodBeforeAdviceInterceptor, AfterReturningAdviceInterceptor, ThrowsAdviceInterceptor 등의 구현체는 어라운드 어드바이스인 MethodInterceptor를 구현하면서 조인 포인트인 MethodInvocation의 proceed()을 필요에 맞게 알맞은 위치에서 호출하고 있다.

invoke() 메서드의 MethodInvocation 인자는 호출되는 메서드, 대상 조인 포인트, AOP 프록시 및 메서드에 대한 인자를 노출한다.  
invoke() 메서드는 호출 결과를 반환해야 한다.  
즉, 조인 포인트의 반환 값이다.  

MethodInterceptor 구현체 예제는 다음과 같다.  

```java
public class DebugInterceptor implements MethodInterceptor {

    public Object invoke(MethodInvocation invocation) throws Throwable {
        System.out.println("Before: invocation=[" + invocation + "]");
        Object rval = invocation.proceed();
        System.out.println("Invocation returned");
        return rval;
    }
}
```

더 간단한 어드바이스 타입은 before 어드바이스이다.  
이는 메서드 진입 전에만 호출되기 때문에 MethodInvocation 객체가 필요하지 않다.  

```java
public interface MethodBeforeAdvice extends BeforeAdvice {

    void before(Method m, Object[] args, Object target) throws Throwable;
}
```

반환 타입이 void임에 주목하자.  
before 어드바이스는 조인 포인트가 실행되기 전에 커스텀한 행위를 수행할 수 있지만, 반환 값을 변경할 수는 없다.  
before 어드바이스에서 예외를 던지면 실행되던 인터셉터 체인을 중지하고, 예외는 인터셉터 체인 위로 전파된다.  

조인 포인트에서 예외가 발생한 경우에는 조인 포인트가 반환된 후에 throws 어드바이스가 호출된다.  
다음과 같은 형태를 가진다.  

```java
afterThrowing([Method, args, target], subclassOfThrowable)
```

마지막 예외 인자만 필수고, 앞에 있는 세 개의 인자는 필요에 따라 받아서 사용할 수 있다.  

after returning 어드바이스는 `org.springframework.aop.AfterReturningAdvice` 인터페이스를 구현해야 한다.  

```java
public interface AfterReturningAdvice extends Advice {

    void afterReturning(Object returnValue, Method m, Object[] args, Object target)
            throws Throwable;
}
```

after returning 어드바이스는 반환 값(수정 불가), 호출된 메서드, 메서드 인자 및 타깃 객체에 접근할 수 있다.  

## AOP 프록시를 생성하기 위한 ProxyFactoryBean 사용

만약 당신이 비즈니스 객체들을 위해 스프링 IoC 컨테이너를 사용한다면, 당신은 스프링 AOP FactoryBean 구현체 중 하나를 사용하길 원할 것이다.  

스프링에서 AOP 프록시를 생성하기 위한 가장 간단한 방법은 `org.springframework.aop.framework.ProxyFactoryBean` 을 사용하는 것이다.  
이는 포인트것, 적용되는 어드바이스, 순서 등의 완전한 제어를 제공한다.  
물론 이러한 제어가 필요하지 않은 경우에도 옵션이 존재한다.  

### 기본

다른 스프링 FactoryBean 구현체와 마찬가지로 ProxyFactoryBean은 간접적인 레벨을 도입한다.  
foo라는 이름으로 ProxyFactoryBean을 정의하면 foo를 참조하는 객체는 ProxyFactoryBean 인스턴스 자체를 보지 않고, getObject() 메서드에 의해 생성된 객체를 바라보게 된다.  
이 메서드는 타깃 객체를 래핑하는 AOP 프록시 객체를 만든다.  

ProxyFactoryBean의 또 다른 중요한 장점은 어드바이스와 포인트컷이 IoC에 의해 관리된다는 것이다.  
이는 매우 강력한 기능으로, 다른 AOP 프레임워크에서는 하기 힘든 특정 접근 방식을 가능하게 한다.  
예를 들어, 어드바이스가 애플리케이션 객체를 참조할 수 있으며, DI의 이점을 누릴 수 있다.  

### 자바빈 프로퍼티

### JDK와 CGLIB 기반 프록시

---

## Auto-proxy 기능 사용

### Auto-proxy 빈 정의

선택한 빈 정의를 자동으로 프록시할 수 있는 auto-proxy 빈 정의  

컨테이너가 로드될 때 모든 빈 정의를 수정할 수 있는 빈 후처리기 인프라를 기반으로 한다.  

- BeanNameAutoProxyCreator
    - 와일드카드 등으로 매칭되는 빈 이름을 보고 AOP 프록시를 자동 생성
- DefaultAdvisorAutoProxyCreator
    - 빈 이름을 등록할 필요 없이 컨텍스트에서 적합한 어드바이저를 자동으로 적용  
    각 어드바이저에 포함된 포인트컷을 자동으로 평가하여 어떤 어드바이스가 각 비즈니스 객체에 포함되어야 하는지 확인한다.
    - 많은 비즈니스 객체에 동일한 어드바이스를 일관되게 적용하려는 경우 유용하다.
    - 필터링과 순서 지정 지원

## TargetSource 구현체 사용

스프링은 `org.springframework.aop.TargetSource` 인터페이스에 표현된 TargetSource의 개념을 제공한다.  
이 인터페이스는 조인 포인트를 구현하는 대상 객체를 반환하는 역할을 한다.  

### Hot-swappable 타깃 소스

`org.springframework.aop.target.HotSwappableTargetSource` 는 AOP 프록시의 타깃을 전환하는 동시에 호출자가 참조를 유지하도록 하기 위해 존재한다.  

```java
HotSwappableTargetSource swapper = (HotSwappableTargetSource) beanFactory.getBean("swapper");
Object oldTarget = swapper.swap(newTarget);
```

## 새로운 어드바이스 타입 정의

스프링 AOP는 확장 가능하도록 설계되었다.  
around, before, throws, after returning 어드바이스 외에도 임의의 어드바이스 타입을 지원하는 것이 가능하다.  

`org.springframework.aop.framework.adapter` 패키지는 핵심 프레임워크를 변경하지 않고 새로운 커스텀 타입에 대한 지원을 추가할 수 있는 SPI 패키지이다.  
커스텀 어드바이스 타입에 대한 유일한 제약은 `org.aopalliance.aop.Advice` 마커 인터페이스를 구현해야 한다는 것이다.  

## Null-safety

자바는 타입 시스템으로 null 안전성을 표현하는 것을 허용하지 않지만 스프링 프레임워크는 다음 어노테이션을 제공하여 API 및 필드의 null 허용 가능성을 선언할 수 있다.  

- `@Nullable`
    - 특정 매개변수, 반환 값 또는 필드가 null일 수 있음
- `@NonNull`
    - 특정 매개변수, 반환 값 또는 필드가 null일 수 없음
- `@NonNullApi`
    - 매개변수 및 반환 값에 대한 기본 상태가 null이 아님을 선언하는 패키지 수준의 어노테이션
- `@NonNullFields`
    - 필드의 기본 상태가 null이 아닌 것을 선언하는 패키지 수준의 어노테이션

## 데이터 버퍼와 코덱

스프링 코어 모듈은 다음과 같이 다양한 바이트 버퍼 API와 함께 동작하는 추상화 세트를 제공한다.  

- DataBufferFactory
    - 데이터 버퍼 생성을 추상화
- DataBuffer
    - 풀링될 수 있는 바이트 버퍼
- DataBufferUtils
    - 데이터 버퍼에 대한 유틸 메서드 제공
- Codecs
    - 스트림 데이터 버퍼 스트림을 더 높은 수준의 객체로 디코딩하거나 인코딩합니다.
