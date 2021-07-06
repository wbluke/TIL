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

## 스프링 타입 변환

스프링3에서는 타입 변환을 위한 `core.convert` 패키지가 소개되었다.  
스프링 컨테이너 내에서 이 시스템을 PropertyEditor 구현의 대안으로 사용하여 빈 속성 값 문자열을 필수 타입으로 변환할 수 있다.  

### Converter SPI

타입 변환을 구현하는 SPI는 간단하고 강력한 형식이다.  

```java
package org.springframework.core.convert.converter;

public interface Converter<S, T> {

    T convert(S source);
}
```

Converter를 만들고 싶다면, Converter 인터페이스를 구현하고 변환 전 타입인 S와 변환할 타입인 T를 지정해주면 된다.  
마찬가지로 S 배열을 컬렉션 T로 변환하고 싶은 경우에도, Converter를 적용할 수 있다.  

`convert(S)` 호출 시 인자인 S는 null이 아님이 보장되어야만 한다.  
Converter는 변환 실패 시 언체크 예외를 던질 것이다.  
특별히, 유효하지 않은 값에 대해서는 IllegalArgumentException을 던져야 한다.  
또한 Converter는 항상 스레드 안전해야함을 명심하자.  

`core.convert.support` 패키지에 있는 여러 컨버터도 편의를 위해 제공된다.  
예를 들어 StringToInteger 같은 경우는 다음과 같다.  

```java
package org.springframework.core.convert.support;

final class StringToInteger implements Converter<String, Integer> {

    public Integer convert(String source) {
        return Integer.valueOf(source);
    }
}
```

### ConverterFactory 사용

만약 계층 구조의 전체 클래스를 대상으로 변환 로직을 중앙화하고 싶은 경우, ConverterFactory를 사용할 수 있다.  

```java
package org.springframework.core.convert.converter;

public interface ConverterFactory<S, R> {

    <T extends R> Converter<S, T> getConverter(Class<T> targetType);
}
```

S는 변환하기 전 타입, R은 변환하고자 하는 클래스의 범위(range)를 지정하면 된다.  
`getConverter(Class<T>)` 의 T는 R의 서브클래스이다.  

StringToEnumConverterFactory 예시는 다음과 같다.  

```java
package org.springframework.core.convert.support;

final class StringToEnumConverterFactory implements ConverterFactory<String, Enum> {

    public <T extends Enum> Converter<String, T> getConverter(Class<T> targetType) {
        return new StringToEnumConverter(targetType);
    }

    private final class StringToEnumConverter<T extends Enum> implements Converter<String, T> {

        private Class<T> enumType;

        public StringToEnumConverter(Class<T> enumType) {
            this.enumType = enumType;
        }

        public T convert(String source) {
            return (T) Enum.valueOf(this.enumType, source.trim());
        }
    }
}
```

### GenericConverter 사용

만약 정교한 Converter 구현체가 필요하다면, GenericConverter 인터페이스를 고려해보면 좋다.  
유연하지만 강한 타입은 아닌 Converter 대신, GenericConverter는 여러 개의 소스와 타겟 간 타입 변환을 지원한다.  

```java
package org.springframework.core.convert.converter;

public interface GenericConverter {

    public Set<ConvertiblePair> getConvertibleTypes();

    Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);
}
```

GenericConverter를 구현하면, `getConvertibleTypes()` 는 지원하는 소스 → 타겟 타입 페어를 반환한다.  
`convert(Object, TypeDescriptor, TypeDescriptor)` 에는 변환 로직을 구현하면 된다.  

GenericConverter의 좋은 예시는 자바의 배열과 컬렉션 간 변환이다.  
ArrayToCollectionConverter는 소스 배열이 타겟 컬렉션으로 변환되도록 해준다.  

> GenericConverter는 좀 더 복잡한 SPI 인터페이스이기 때문에, 꼭 필요할 때만 사용하면 된다.  
보통은 Converter나 ConverterFactory로 사용한다.

가끔은, Converter가 특정 조건에서만 작동하기를 바라는 경우가 있다.  
예를 들어, 어떤 타겟 필드에 특정 어노테이션이 붙어있을 때에만 변환하기를 바라거나, 어떤 클래스에 특정 메서드가 구현되어 있을 경우에만 변환하기를 바랄 수 있다.  
ConditionalGenericConverter는 이러한 것들을 가능하게 하는 GenericConverter와 ConditionalConverter 인터페이스의 조합이다.  

```java
public interface ConditionalConverter {

    boolean matches(TypeDescriptor sourceType, TypeDescriptor targetType);
}

public interface ConditionalGenericConverter extends GenericConverter, ConditionalConverter {
}
```

ConditionalGenericConverter의 좋은 예시는 영속화된 엔티티의 식별자와 엔티티 참조 간을 변환하는 IdToEntityConverter이다.  
IdToEntityConverter는 타겟 엔티티 타입이 정적 finder 메서드를 선언한 경우에만 변환을 진행한다.  

### ConversionService API

ConversionService는 런타임에 타입 변환을 실행하기 위한 통합 API를 제공한다.  

```java
package org.springframework.core.convert;

public interface ConversionService {

    boolean canConvert(Class<?> sourceType, Class<?> targetType);

    <T> T convert(Object source, Class<T> targetType);

    boolean canConvert(TypeDescriptor sourceType, TypeDescriptor targetType);

    Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);
}
```

대부분의 ConversionService 구현체는 ConverterRegistry도 구현하는데, converter를 등록하기 위한 SPI를 제공하기 위해서이다.  
내부적으로 ConversionService 구현체는 타입 변환 로직을 등록된 converter에 위임한다.  

ConversionService는 `core.convert.support` 패키지에서 제공된다.  
GenericConversionService는 대부분의 환경에서 사용되기 위한 일반적인 목적의 구현체이다.  
ConversionServiceFactory는 ConversionService를 생성하기 위한 편리한 팩토리를 제공한다.  

### ConversionService 구성

ConversionService는 애플리케이션 시작 시 초기화되고, 여러 스레드에 의해 공유되도록 만들어진 무상태 객체이다.  
스프링 애플리케이션에서는 전형적으로 스프링 컨테이너마다 ConversionService를 구성해야 한다.  
스프링은 프레임워크에 의한 타입 변환이 필요한 경우 ConversionService를 사용한다.  
또한 ConversionService를 빈에 직접 주입해서 사용할 수도 있다.  

> 만약 어떤 ConversionService도 스프링에 등록되지 않는다면, 기존의 PropertyEditor 기반 시스템이 사용된다.

### 프로그래밍 방식으로 ConversionService 사용

ConversionService를 프로그래밍 방식으로 사용하고자 한다면, 다음과 같이 빈에 주입해서 사용할 수 있다.  

```java
@Service
public class MyService {

    public MyService(ConversionService conversionService) {
        this.conversionService = conversionService;
    }

    public void doIt() {
        this.conversionService.convert(...)
    }
}
```

대부분의 경우 타겟 타입으로의 변환을 위해 `convert()` 메서드를 사용하겠지만, 이는 매개변수화된 컬렉션과 같은 복잡한 타입에서는 동작하지 않는다.  
예를 들어 정수 리스트를 문자열 리스트로 변환하고자 할 때, 소스와 타겟 타입에 대한 정의가 필요하다.  

다행히도, TypeDescriptor는 이런 상황을 해결할 수 있는 옵션을 제공한다.  

```java
DefaultConversionService cs = new DefaultConversionService();

List<Integer> input = ...
cs.convert(input,
    TypeDescriptor.forObject(input), // List<Integer> type descriptor
    TypeDescriptor.collection(List.class, TypeDescriptor.valueOf(String.class)));
```

DefaultConversionService는 대부분의 환경에 적합한 converter를 자동으로 등록한다.  
여기에는 컬렉션 converter, 스칼라 converter 및 기본 Object-String converter가 포함된다.  
DefaultConversionService 클래스에서 정적 addDefaultConverters 메서드를 사용하여 ConverterRegistry에 converter를 등록 할 수 있다.  

## 스프링 필드 포맷팅

이제 웹이나 데스크탑 환경과 같은 일반적인 클라이언트 환경에서의 타입 변환을 생각해 보자.  
이런 환경에서는, 문자열을 필요한 타입으로 변환하고, 다시 뷰 렌터링을 위해 문자열로 변환하기도 한다.  
Converter SPI는 이러한 형식 요구 사항을 직접 해결하지 않는다.  
이를 해결하기 위해 스프링 3은 클라이언트 환경을 위한 PropertyEditor 구현체의 강력한 대안으로 편리한 Formatter SPI를 도입했다.  

일반적으로 범용적인 목적의 타입 변환 로직(예를 들어, `java.util.Date` 와 Long 간 변환)을 구현해야 할 때 Converter SPI를 사용할 수 있다.  
그리고 클라이언트 환경에서 작업하고 현지화된 필드 값을 파싱하고 출력해야 하는 경우 Formatter SPI를 사용할 수 있다.  
ConversionService는 두 SPI에 대한 통합 타입 API를 제공한다.  

### Formatter SPI

Formatter SPI는 간단하고 강력한 포맷팅 로직을 제공한다.  

```java
package org.springframework.format;

public interface Formatter<T> extends Printer<T>, Parser<T> {
}
```

Formatter는 Printer와 Parser를 확장한다.  

```java
public interface Printer<T> {

    String print(T fieldValue, Locale locale);
}
```

```java
import java.text.ParseException;

public interface Parser<T> {

    T parse(String clientValue, Locale locale) throws ParseException;
}
```

Formatter를 만들려면, Formatter 인터페이스를 구현한다.  
매개변수 T는 Date와 같이 포맷팅하려는 타입 객체이다.  
print() 메서드는 T를 클라이언트 환경에 보여주기 위한 메서드이고, parse() 메서드는 반대로 포맷팅된 표현을 T 타입 인스턴스로 변환하기 위한 메서드이다.  
파싱에 실패하면 ParseException이나 IllegalArgumentException을 던져야 한다.  
Formatter 구현체는 스레드 안전해야 함을 명심하자.  

DateFormatter의 예시는 다음과 같다.  

```java
package org.springframework.format.datetime;

public final class DateFormatter implements Formatter<Date> {

    private String pattern;

    public DateFormatter(String pattern) {
        this.pattern = pattern;
    }

    public String print(Date date, Locale locale) {
        if (date == null) {
            return "";
        }
        return getDateFormat(locale).format(date);
    }

    public Date parse(String formatted, Locale locale) throws ParseException {
        if (formatted.length() == 0) {
            return null;
        }
        return getDateFormat(locale).parse(formatted);
    }

    protected DateFormat getDateFormat(Locale locale) {
        DateFormat dateFormat = new SimpleDateFormat(this.pattern, locale);
        dateFormat.setLenient(false);
        return dateFormat;
    }
}
```

### 어노테이션 기반 포맷팅

필드 포맷팅은 필트 타입이나 어노테이션으로 구성될 수 있다.  
어노테이션을 Formatter로 바인딩하려면, AnnotationFormatterFactory를 구현한다.  

```java
package org.springframework.format;

public interface AnnotationFormatterFactory<A extends Annotation> {

    Set<Class<?>> getFieldTypes();

    Printer<?> getPrinter(A annotation, Class<?> fieldType);

    Parser<?> getParser(A annotation, Class<?> fieldType);
}
```

구현체를 만들려면, 매개변수 A에 포맷팅 로직과 관련시킬 어노테이션 타입을 적용하면 된다. (예 : DateTimeFormat)  
그리고 getPrinter(), getParser() 각 메서드에서 필요한 Printer와 Parser를 반환하도록 구현하면 된다.  

다음 예시는 `@NumberFormat` 어노테이션에 대한 AnnotationFormatterFactory 구현체이다.  

```java
public final class NumberFormatAnnotationFormatterFactory
        implements AnnotationFormatterFactory<NumberFormat> {

    public Set<Class<?>> getFieldTypes() {
        return new HashSet<Class<?>>(asList(new Class<?>[] {
            Short.class, Integer.class, Long.class, Float.class,
            Double.class, BigDecimal.class, BigInteger.class }));
    }

    public Printer<Number> getPrinter(NumberFormat annotation, Class<?> fieldType) {
        return configureFormatterFrom(annotation, fieldType);
    }

    public Parser<Number> getParser(NumberFormat annotation, Class<?> fieldType) {
        return configureFormatterFrom(annotation, fieldType);
    }

    private Formatter<Number> configureFormatterFrom(NumberFormat annotation, Class<?> fieldType) {
        if (!annotation.pattern().isEmpty()) {
            return new NumberStyleFormatter(annotation.pattern());
        } else {
            Style style = annotation.style();
            if (style == Style.PERCENT) {
                return new PercentStyleFormatter();
            } else if (style == Style.CURRENCY) {
                return new CurrencyStyleFormatter();
            } else {
                return new NumberStyleFormatter();
            }
        }
    }
}
```

다음과 같은 형식으로 사용 가능하다.  

```java
public class MyModel {

    @NumberFormat(style=Style.CURRENCY)
    private BigDecimal decimal;

    @DateTimeFormat(iso=ISO.DATE)
    private Date date;
}
```

### FormatterRegistry SPI

FormatterRegistry는 formatter와 converter를 등록하기 위한 SPI이다.  
FormatterConversionService는 대부분의 환경에서 사용하기 적합한 FormatterRegistry 구현체이다.  

```java
package org.springframework.format;

public interface FormatterRegistry extends ConverterRegistry {

    void addPrinter(Printer<?> printer);

    void addParser(Parser<?> parser);

    void addFormatter(Formatter<?> formatter);

    void addFormatterForFieldType(Class<?> fieldType, Formatter<?> formatter);

    void addFormatterForFieldType(Class<?> fieldType, Printer<?> printer, Parser<?> parser);

    void addFormatterForFieldAnnotation(AnnotationFormatterFactory<? extends Annotation> annotationFormatterFactory);
}
```

FormatterRegistry SPI를 사용하면 컨트롤러 전체에 중복 설정을 하는 대신, 포맷팅 규칙을 중앙화할 수 있다.  
예를 들어 모든 날짜 필드가 특정 방식으로 포맷팅되거나 특정 어노테이션이 있는 필드가 특정 방식으로 포맷팅되도록 할 수 있다.  

### FormatterRegistrar SPI

FormatterRegistrar는 FormatterRegistry를 통해 formatter 및 converter를 등록하기 위한 SPI이다.  

```java
package org.springframework.format;

public interface FormatterRegistrar {

    void registerFormatters(FormatterRegistry registry);
}
```

FormatterRegistrar는 날짜 형식과 같은 포맷팅 카테고리에 대해 여러 관련된 converter와 formatter를 등록할 때 유용하다.  

## 전역 날짜와 시간 포맷 설정

기본적으로 `@DateTimeFormat` 을 사용하지 않은 날짜와 시간은 DateFormat.SHORT를 사용하여 문자열로 변환된다.  
원하는 경우 전역 포맷을 지정하여 변경할 수 있다.  
그렇게 하려면, 스프링이 기본 포맷터를 등록하지 않음을 보장해야 한다.  
다음 클래스의 도움을 받으면 된다.  

- org.springframework.format.datetime.standard.DateTimeFormatterRegistrar
- org.springframework.format.datetime.DateFormatterRegistrar

아래 예제는 전역적인 `yyyyMMdd` 포맷을 등록하는 설정 예제이다.  

```java
@Configuration
public class AppConfig {

    @Bean
    public FormattingConversionService conversionService() {

        // DefaultFormattingConversionService 사용, 기본 등록은 제외
        DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService(false);

        // @NumberFormat 지원은 보장
        conversionService.addFormatterForFieldAnnotation(new NumberFormatAnnotationFormatterFactory());

        // 특정 전역 포맷을 사용한 JSR-310 날짜 변환 등록
        DateTimeFormatterRegistrar registrar = new DateTimeFormatterRegistrar();
        registrar.setDateFormatter(DateTimeFormatter.ofPattern("yyyyMMdd"));
        registrar.registerFormatters(conversionService);

        // 특정 전역 포맷을 사용한 날짜 변환 등록
        DateFormatterRegistrar registrar = new DateFormatterRegistrar();
        registrar.setFormatter(new DateFormatter("yyyyMMdd"));
        registrar.registerFormatters(conversionService);

        return conversionService;
    }
}
```

## 자바 빈 유효성 검사

### 빈 유효성 검사 개요

빈 유효성 검사는 제약 선언과 메타 데이터를 통한 일반적인 유효성 검사를 제공한다.  
이를 사용하려면, 도메인 모델 속성에 런타임 시점에 동작하는 유효성 검사 제약을 선언하면 된다.  
기본 제공되는 제약도 있고, 커스텀하게 지정할 수도 있다.  

예를 들어 빈 유효성 검사를 제공한 PersonForm 클래스는 다음과 같다.  

```java
public class PersonForm {

    @NotNull
    @Size(max=64)
    private String name;

    @Min(0)
    private int age;
}
```

### 빈 유효성 검사 제공자 구성

스프링은 빈 유효성 검사 제공자를 포함한 빈 유효성 검사 API를 전격 지원한다.  
`javax.validation.ValidatorFactory` 또는 `javax.validation.Validator` 를 주입해서 사용할 수 있다.  

기본 Validator를 스프링 빈으로 구성하기 위해 LocalValidatorFactoryBean을 사용할 수 있다.  

```java
import org.springframework.validation.beanvalidation.LocalValidatorFactoryBean;

@Configuration
public class AppConfig {

    @Bean
    public LocalValidatorFactoryBean validator() {
        return new LocalValidatorFactoryBean();
    }
}
```

LocalValidatorFactoryBean은 `javax.validation.ValidatorFactory` 와 `javax.validation.Validator` 를 구현하고, 스프링의 `org.springframework.validation.Validator` 도 구현한다.  
유효성 검사 로직이 필요한 빈에 이러한 인터페이스 중 하나에 대한 참조를 주입할 수 있다.  

빈 유효성 검사 API 사용을 원한다면, `javax.validation.Validator` 를 주입받을 수 있다.  

```java
import javax.validation.Validator;

@Service
public class MyService {

    @Autowired
    private Validator validator;
}
```

스프링 유효성 검사 API가 필요한 빈이라면, `org.springframework.validation.Validator` 를 주입할 수도 있다.  

```java
import org.springframework.validation.Validator;

@Service
public class MyService {

    @Autowired
    private Validator validator;
}
```

각각의 빈 유효성 검사는 다음 2가지로 구성된다.  

- 제약과 그 속성값을 선언하는 `@Constraint` 어노테이션
- 제약의 행동을 구현하는 `javax.validation.ConstraintValidator` 구현체

선언을 구현부와 연결하기 위해 @Constraint 어노테이션은 해당하는 ConstraintValidator 구현체를 참조한다.  
런타임 시 ConstraintValidatorFactory는 도메인 모델에서 해당 어노테이션이 발견될 때 참조된 구현체를 인스턴스화한다.  

기본적으로 LocalValidatorFactoryBean은 스프링을 사용하여 ConstraintValidator 인스턴스를 만드는 SpringConstraintValidatorFactory를 구성한다.  
이렇게 하면 커스텀 ConstraintValidators가 다른 스프링 빈과 마찬가지로 의존성 주입의 이점을 얻을 수 있다.  

```java
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy=MyConstraintValidator.class)
public @interface MyConstraint {
}
```

```java
import javax.validation.ConstraintValidator;

public class MyConstraintValidator implements ConstraintValidator {

    @Autowired;
    private Foo aDependency;

    // ...
}
```

### DataBinder 설정

스프링3 부터, Validator와 함께 DataBinder 인스턴스를 구성할 수 있다.  
구성한 후 binder.validate() 메서드를 호출하여 Validator를 사용할 수 있다.  
발생한 유효성 에러는 binder의 BindingResult에 추가된다.  

```java
Foo target = new Foo();
DataBinder binder = new DataBinder(target);
binder.setValidator(new FooValidator());

// 타겟 객체 바인딩
binder.bind(propertyValues);

// 타겟 객체 유효성 검사
binder.validate();

// 유효성 에러를 가진 BindingResult
BindingResult results = binder.getBindingResult();
```

dataBinder.addValidators(), dataBinder.replaceValidators() 를 사용하여 다수의 Validator 인스턴스를 등록할 수도 있다.  
이는 전역적으로 구성된 빈 유효성 검사를 DataBinder 인스턴스에 로컬로 구성된 스프링 Validator와 결합할 때 유용하다.
