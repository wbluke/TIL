비즈니스 로직으로 유효성 검사를 고려하는 데는 장단점이 있으며, 스프링은 둘 중 하나도 배제하지 않는 유효성 검사 설계를 제공한다.  
유효성 검사는 웹 계층에 종속적이어서도 안 되며, 현지화가 쉬워야 하고, 어떤 validator도 적용할 수 있어야 한다.  
이러한 것들을 고려하여, 스프링은 애플리케이션의 모든 계층에서 기본적이고 탁월한 사용이 가능한 Validator를 제공한다.  

데이터 바인딩은 사용자 입력을 애플리케이션 도메인 모델에 동적으로 바인딩하는 데 유용하다.  
스프링은 이를 위해 적절한 이름의 DataBinder를 제공한다.  
Validator와 DataBinder는 웹 계층에서 주로 사용되지만 국한되지는 않는 validation 패키지를 구성한다.  

BeanWrapper는 스프링의 기본 개념이며 많은 곳에서 사용된다.  
하지만 BeanWrapper를 직접 사용할 일이 없을 것인데, 공식 문서이다보니 약간은 설명할 필요를 느꼈다.  
만약 BeanWrapper를 사용할 일이 있다면, 데이터를 객체에 반영하려고 할 때일 것이다.  

스프링의 DataBinder와 낮은 수준의 BeanWrapper는 둘 다 프로퍼티 값들을 파싱하고 포맷팅하기 위해  PropertyEditorSupport 구현체를 사용한다.  

## 스프링 Validator 인터페이스를 사용한 유효성 검사

스프링은 객체의 유효성을 검사하기 위해 Validator 인터페이스를 제공한다.  
Validator 인터페이스는 Errors 객체를 사용하는데, 유효성 검사 시 validator들은 유효성 검사 실패 건을 Errors 객체에 기록한다.  

만약 String name과 int age를 가지는 Person 객체가 있을 때, 다음과 같이 적용할 수 있다.  
Validator 는 2개의 메서드를 구현해야 한다.  

- `supports(Class)`
    - 특정 Class를 지원하는지 여부
- `validate(Object, org.springframework.validation.Errors)`
    - 유효성 검사 및 Errors 객체에 에러 등록

```java
public class PersonValidator implements Validator {

    /**
     * This Validator validates only Person instances
     */
    public boolean supports(Class clazz) {
        return Person.class.equals(clazz);
    }

    public void validate(Object obj, Errors e) {
        ValidationUtils.rejectIfEmpty(e, "name", "name.empty");
        Person p = (Person) obj;
        if (p.getAge() < 0) {
            e.rejectValue("age", "negativevalue");
        } else if (p.getAge() > 110) {
            e.rejectValue("age", "too.darn.old");
        }
    }
}
```

ValidationUtils의 rejectIfEmpty() 메서드는 name 속성이 null이거나 빈 문자열이면 거절하는 메서드이다.  

조금 더 복잡한 경우에, 예를 들어 Customer 클래스 내에 Address 객체가 있는 경우 단일 Validator 클래스를 구현해서 해결할 수도 있지만, 각각의 Validator를 만들어서 유효성 검사 로직을 캡슐화하는 것이 더 좋다.  

```java
public class CustomerValidator implements Validator {

    private final Validator addressValidator;

    public CustomerValidator(Validator addressValidator) {
        if (addressValidator == null) {
            throw new IllegalArgumentException("The supplied [Validator] is " +
                "required and must not be null.");
        }
        if (!addressValidator.supports(Address.class)) {
            throw new IllegalArgumentException("The supplied [Validator] must " +
                "support the validation of [Address] instances.");
        }
        this.addressValidator = addressValidator;
    }

    /**
     * 이 Validator는 Customer 인스턴스 뿐만 아니라 그 서브클래스들도 검증한다.
     */
    public boolean supports(Class clazz) {
        return Customer.class.isAssignableFrom(clazz);
    }

    public void validate(Object target, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "firstName", "field.required");
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "surname", "field.required");
        Customer customer = (Customer) target;
        try {
            errors.pushNestedPath("address");
            ValidationUtils.invokeValidator(this.addressValidator, customer.getAddress(), errors);
        } finally {
            errors.popNestedPath();
        }
    }
}
```

## 에러 메시지 코드 분석

MessageSource를 사용하여 에러 메시지를 출력하려면 필드를 거부할 때 제공하는 에러 코드(위 예시의 name 이나 age)를 사용하면 된다.  
예를 들어 ValidationUtils 클래스를 사용하여 rejectValue를 호출하거나 Errors 인터페이스의 다른 reject 메서드 중 하나를 호출하면 우리가 전달한 코드를 등록할 뿐만 아니라 여러 추가적인 에러 코드도 같이 전달한다.  
MessageCodesResolver는 Errors 인터페이스가 등록하는 에러 코드를 결정한다.  
기본적으로는 DefaultMessageCodesResolver가 사용된다.  

## 빈 조작과 BeanWrapper

Bean 패키지에서 가장 중요한 클래스 중 하나는 BeanWrapper 인터페이스와 그 구현체이다.  
BeanWrapper는 속성 값을 설정(set)하거나 가져오고(get), 속성을 쿼리하여 읽기 또는 쓰기 가능 여부를 결정하는 기능을 제공한다.  
또한 BeanWrapper는 중첩된 속성에 대한 지원을 제공하여 하위 속성의 속성값을 무제한 깊이로 설정할 수 있게 한다.  
BeanWrapper는 일반적으로 애플리케이션 코드에서 직접적으로 사용되지는 않지만 DataBinder 및 BeanFactory에서 사용된다.  

```java
BeanWrapper company = new BeanWrapperImpl(new Company());
// 회사 이름 설정
company.setPropertyValue("name", "Some Company Inc.");
// ... 이렇게도 가능하다.
PropertyValue value = new PropertyValue("name", "Some Company Inc.");
company.setPropertyValue(value);

// 좋다, 디렉터를 생성하고 회사에 설정해보자.
BeanWrapper jim = new BeanWrapperImpl(new Employee());
jim.setPropertyValue("name", "Jim Stravinsky");
company.setPropertyValue("managingDirector", jim.getWrappedInstance());

// 회사를 통해 managingDirector의 급여를 조회
Float salary = (Float) company.getPropertyValue("managingDirector.salary");
```

### 내장된 PropertyEditor 구현체들

스프링은 Object와 String 간 변환을 위해 PropertyEditor를 사용한다.  
이는 객체 자체 대신 다른 방식으로 쉽게 속성 값들을 표현할 수 있도록 한다.  
예를 들어, Date 객체는 인간친화적인 방식('2007-12-09'와 같은 String)으로 표현될 수 있고, 그 반대도 가능하다.  

내장된 구현체들은 다음과 같다.  

- ByteArrayPropertyEditor
- ClassEditor
- CustomBooleanEditor
- CustomCollectionEditor
- CustomDateEditor
- CustomNumberEditor
- FileEditor
- InputStreamEditor
- LocaleEditor
- PatternEditor
- PropertiesEditor
- StringTrimmerEditor
- URLEditor

이런 것들은 커스텀한 PropertyEditor 타입을 등록해서도 구성할 수 있다.  

```java
package example;

public class ExoticTypeEditor extends PropertyEditorSupport {

    public void setAsText(String text) {
        setValue(new ExoticType(text.toUpperCase()));
    }
}
```

PropertyEditorRegistrar를 이용해서 다음과 같이 커스텀 속성 에디터를 등록할 수 있다.  

```java
package com.foo.editors.spring;

public final class CustomPropertyEditorRegistrar implements PropertyEditorRegistrar {

    public void registerCustomEditors(PropertyEditorRegistry registry) {

        // it is expected that new PropertyEditor instances are created
        registry.registerCustomEditor(ExoticType.class, new ExoticTypeEditor());

        // you could register as many custom property editors as are required here...
    }
}
```
