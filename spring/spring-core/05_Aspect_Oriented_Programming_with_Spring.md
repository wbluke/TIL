관점 지향 프로그래밍(AOP)는 프로그램 구조에 대한 다른 생각을 제공함으로써 객체 지향 프로그래밍(OOP)을 보완한다.  
OOP에서 모듈화의 핵심 단위는 클래스인 반면, AOP에서 모듈화의 단위는 애스펙트이다.  
애스펙트는 여러 타입과 객체를 관통하는 (트랜잭션 관리 같은) 개념의 모듈화를 가능하게 한다.  

스프링의 핵심 컴포넌트 중 하나가 바로 AOP 프레임워크이다.  
스프링 IoC 컨테이너가 AOP에 의존하지 않는 반면에 (즉, AOP를 사용하고 싶지 않으면 사용하지 않을 수 있다), AOP는 아주 유용한 미들웨어 해결책으로 스프링 IoC를 보완한다.  

AOP는 스프링 프레임워크에서 다음을 위해 사용된다.  

- 선언적 엔터프라이즈 서비스를 제공한다. 이러한 서비스 중 가장 중요한 것은 선언적 트랜잭션 관리이다.
- 사용자가 커스텀한 애스펙트를 구현하여 AOP와 함께 OOP 사용을 보완할 수 있다.

## AOP 개념

몇 가지 중심적인 AOP 개념과 용어를 정의해 보자.  
이 용어들은 스프링에 국한된 용어는 아니다.  
불행히도 AOP 용어가 직관적이지는 않지만, 스프링이 자체 용어를 사용하는 것보다는 덜 혼란스러울 것이다.  

- 애스펙트(Aspect)
    - 여러 클래스를 관통하는 개념의 모듈화이다.  
    트랜잭션 관리가 크로스컷팅 개념의 좋은 예시다.  
    스프링 AOP에서 애스펙트는 일반 클래스에 의해 구현되거나 `@Aspect` 어노테이션에 의해 마킹된 일반 클래스에 의해 구현된다.
- 조인 포인트(Join point)
    - 메서드의 실행이나 예외 처리 같은 프로그램 실행 포인트.  
    스프링 AOP에서 조인 포인트는 항상 메서드 실행을 의미한다.
- 어드바이스(Advice)
    - 특정 조인 포인트에서 애스펙트에 의해 행해지는 행위.  
    어드바이스의 다양한 타입은 "around", "before", "after" 어드바이스를 포함한다.  
    스프링을 포함한 많은 AOP 프레임워크는 어드바이스를 인터셉터로 모델링하고 조인 포인트에 여러 인터셉터의 체인을 유지한다.
- 포인트컷(Pointcut)
    - 조인 포인트에 매칭되는 서술부.  
    어드바이스는 포인트컷 표현식과 연관되며 포인트컷과 매칭되는 모든 조인 포인트에서 실행된다.  
    포인트컷 표현식에 의해 매칭되는 조인 포인트의 개념은 AOP의 핵심이고 스프링은 기본적으로 AspectJ 포인트컷 표현식 언어를 사용한다.
- 인트로덕션(Introduction)
    - 타입을 대신하여 추가 메서드나 필드를 선언한다.  
    스프링 AOP를 사용하면 어드바이스된 객체에 추가적인 인터페이스(및 해당 구현)를 도입할 수 있다.
- 타깃 객체
    - 하나 이상의 애스펙트에게 어드바이스를 받는 객체.  
    "어드바이스된 객체"라고도 한다.  
    스프링 AOP가 런타임 프록시를 사용하여 구현되기 때문에, 이 객체는 항상 프록시 객체이다.
- AOP 프록시
    - 애스펙트 계약을 구현하기 위해 AOP 프레임워크에 의해 생성된 객체.  
    스프링 프레임워크에서 AOP 프록시는 JDK 다이나믹 프록시이거나 CGLIB 프록시이다.
- 위빙(Weaving)
    - 애스펙트를 다른 애플리케이션 타입 또는 객체와 연결해서 새로운 어드바이스된 객체를 만든다.  
    이는 컴파일 타임, 로드 타임, 런타임에 수행할 수 있다.  
    스프링 AOP는 다른 순수 자바 AOP 프레임워크와 마찬가지로 런타임에 위빙을 수행한다.

스프링 AOP는 다음과 같은 어드바이스 타입을 가진다.  

- Before 어드바이스
    - 조인 포인트 전에 수행되는 어드바이스.  
    그러나 (예외를 던지지 않는 한) 조인 포인트로 진행되는 흐름을 막을 능력은 없다.
- After returning 어드바이스
    - 조인 포인트가 일반적으로 완료된 후에 실행되는 어드바이스 (예외를 던지지 않고 메서드가 완료된 경우)
- After throwing 어드바이스
    - 메서드가 예뢰를 발생시킨 경우 실행되는 어드바이스
- After (finally) 어드바이스
    - 조인 포인트가 종료되는 방식(정상이거나 예외를 던지거나)에 상관 없이 실행하는 어드바이스
- Around 어드바이스
    - 조인 포인트를 감싼 어드바이스.  
    이 방식이 가장 강력한 어드바이스이다.  
    Around 어드바이스는 메서드 호출 전후에 커스텀한 행동을 할 수 있다.  
    또한 자체 반환 값을 반환하거나 예외를 던져서 조인 포인트로 진행할지 어드바이스된 메서드 실행을 단축할지 여부를 선택할 책임을 가지고 있다.

Around 어드바이스는 가장 일반적인 경우의 어드바이스이다.  
AspectJ와 같은 스프링 AOP에서 모든 범위의 어드바이스 타입을 제공하는 반면, 우리는 당신이 요구되는 행위를 구현할 수 있는 최소한의 강력한 어드바이스 타입을 사용하기를 권장한다.  
예를 들어, 만약 메서드의 반환 값으로 캐시를 갱신하고 싶다면, 비록 around 어드바이스가 같은 기능을 해낼 수 있을지라도 around 어드바이스 보다는 after returning 어드바이스를 사용하는 것이 좋다.  

## 스프링 AOP의 효용성과 목적

## AOP 프록시

## @AspectJ 지원

### @AspectJ 지원 허용

### Aspect 선언

### 포인트컷 선언

포인트컷은 관심사의 조인 포인트를 결정하고, 어드바이스가 언제 실행될지를 조절할 수 있게 한다.  
스프링 AOP는 스프링 빈을 위한 메서드 실행 조인 포인트만을 지원하며, 그러므로 당신은 포인트컷이 스프링 빈에서 메서드 실행과 매칭된다고 생각할 수 있다.  
포인트컷 선언은 두 가지로 나뉘는데, 이름과 모든 파라미터를 포함하는 시그니처와 관심사가 있는 메서드 실행부를 결정하는 포인트컷 표현식이다.  
AOP의 @AspectJ 어노테이션 스타일에 따르면, 포인트컷 시그니처는 표준 메서드 정의에 의해 제공되며, 포인트컷 표현식은 `@Pointcut` 어노테이션을 사용해서 표현된다.  
(포인트컷 시그니처를 표현하는 메서드는 반드시 void 반환을 해야 한다.)  

예시는 다음과 같다.  

```java
@Pointcut("execution(* transfer(..))") // 포인트컷 표현식
private void anyOldTransfer() {} // 포인트컷 시그니처
```

`@Pointcut` 어노테이션에 의한 포인트컷 표현식은 표준 AspectJ 포인트컷 표현식이다.  

스프링 AOP는 다음 AspectJ 포인트컷 지정자를 지원한다.  

- `execution`
    - 메서드 실행 조인 포인트를 매칭한다. 스프링 AOP에서 가장 우선되는 지정자이다.
- `within`
- `this`
- `target`
- `args`
- `@target`
- `@args`
- `@within`
- `@annotation`

스프링 AOP는 또한 `bean` 이라는 추가 지정자를 지원한다.  
이 지정자를 사용하면 특정 이름의 스프링 빈 또는 스프링 빈 세트(와일드카드 사용 시)에 대한 조인 포인트 일치를 제한할 수 있다.  

당신은 `&&`, `||`, `!` 를 사용하여 여러 포인트컷 표현식을 조합할 수 있다.  

```java
@Pointcut("execution(public * *(..))") // public 메서드
private void anyPublicOperation() {} 

@Pointcut("within(com.xyz.myapp.trading..*)") // trading 모듈 내의 메서드
private void inTrading() {} 

@Pointcut("anyPublicOperation() && inTrading()") // trading 모듈 내의 public 메서드
private void tradingOperation() {}
```

엔터프라이즈급 애플리케이션을 설계하다보면, 개발자들은 특정 연산 집합이나 애플리케이션 모듈을 몇 가지 애스펙트로 관리하기를 원한다.  
이러한 목적을 위해 `CommonPointcuts` 애스펙트를 정의해서 일반적인 포인트컷 표현식을 사용하기를 원한다.  

```java
package com.xyz.myapp;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class CommonPointcuts {

    @Pointcut("within(com.xyz.myapp.web..*)")
    public void inWebLayer() {}

    @Pointcut("within(com.xyz.myapp.service..*)")
    public void inServiceLayer() {}

    @Pointcut("within(com.xyz.myapp.dao..*)")
    public void inDataAccessLayer() {}

    @Pointcut("execution(* com.xyz.myapp..service.*.*(..))")
    public void businessService() {}

    @Pointcut("execution(* com.xyz.myapp.dao.*.*(..))")
    public void dataAccessOperation() {}

}
```
