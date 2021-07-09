스프링 표현 언어(SpEL)는 런타임에 객체 그래프를 탐색하고 조작할 수 있도록 해주는 강력한 표현 언어이다.  

자바 표현 언어는 여러가지(OGNL, MVEL, JBoss EL 등)가 있지만, SpEL은 스프링 프로덕트 전반적으로 사용할 수 있는 좋은 단일 표현 언어로 스프링 커뮤니티에 제공되었다.  

## 평가

다음 코드는 Hello World 라는 문자열 표현을 평가하는 SpEL API를 소개한다.  

```java
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'"); 
String message = (String) exp.getValue(); // Hello World
```

SpEL 클래스와 인터페이스는 `org.springframework.expression` 패키지와 그 하위 패키지인 `spel.support` 에 위치해 있다.  

ExpressionParser 인터페이스는 표현 문자열을 파싱하는 책임을 가진다.  
위 예제에서, 표현 문자열은 따옴표로 둘러쌓인 문자열인다.  
Expression 인터페이스는 표현 문자열을 평가한다.  
평가 시 ParseException이나 EvaluationException이 각각 parser.parseExpression()과 exp.getValue()에서 발생할 수 있다.  

SpEL은 메서드를 호출하거나 속성에 접근하거나 생성자를 호출하는 등의 다양한 기능을 지원한다.  

다음은 concat 메서드를 호출하는 예제이다.  

```java
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'.concat('!')"); 
String message = (String) exp.getValue(); // Hello World!
```

그 다음은 문자열의 속성값인 Bytes 를 호출하는 예제이다.  

```java
ExpressionParser parser = new SpelExpressionParser();

// 'getBytes()' 호출
Expression exp = parser.parseExpression("'Hello World'.bytes"); 
byte[] bytes = (byte[]) exp.getValue();
```

점을 찍는 방식으로 내부 속성에도 접근할 수 있다.  

```java
ExpressionParser parser = new SpelExpressionParser();

// 'getBytes().length' 호출
Expression exp = parser.parseExpression("'Hello World'.bytes.length"); 
int length = (Integer) exp.getValue();
```

다음과 같이 생성자도 호출할 수 있다.  

```java
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("new String('hello world').toUpperCase()"); 
String message = exp.getValue(String.class);
```

`public <T> T getValue(Class<T> desiredResultType)` 제네릭 메서드는 결과 값의 타입 캐스팅을 필요없게 한다.  
만약 타입 변환 중 실패한다면 EvaluationException을 던진다.  

### EvaluationContext 이해

EvaluationContext 인터페이스는 속성, 메서드 또는 필드를 확인하고 형식 변환을 수행하기 위해 표현식을 평가할 때 사용된다.  
스프링은 다음 두 가지 구현체를 제공한다.  

- SimpleEvaluationContext
    - SpEL의 전체 범위가 필요하지 않은 경우
- StandardEvaluationContext
    - SpEL의 전체 기능 제공

SimpleEvaluationContext는 SpEL 언어의 일부분만 지원한다.  
자바 타입 참조, 생성자, 빈 참조 등은 지원하지 않는다.  
또한 표현식에서 속성과 메서드의 지원 레벨을 선택하도록 요구한다.  
기본적으로는 create() 정적 팩토리 메서드가 속성에 대한 읽기 권한을 제공한다.  
또 다음 중 하나 이상의 조합으로 필요한 지원 수준을 구성하는 빌더를 얻을 수 있다.  

- 사용자 지정 PropertyAccessor (reflection 없음)
- 읽기 전용 액세스를 위한 데이터 바인딩 속성
- 읽기 및 쓰기를 위한 데이터 바인딩 속성

```java
class Simple {
    public List<Boolean> booleanList = new ArrayList<Boolean>();
}

Simple simple = new Simple();
simple.booleanList.add(true);

EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();

// "false" 는 문자열로 전달된다.
// SpEL과 변환 서비스는 Boolean임을 인식하고 그에 따라 변환한다.
parser.parseExpression("booleanList[0]").setValue(context, simple, "false");

// b는 false
Boolean b = simple.booleanList.get(0);
```

### Parser 설정

파서 구성 객체를 사용해서 SpEL 표현식 파서를 구성할 수 있다.  
구성 객체는 일부 표현식 컴포넌트의 행동을 제어한다.  
예를 들어, 배열 또는 컬렉션으로 인덱싱하고 지정된 인덱스의 요소가 null이면, SpEL은 자동으로 해당 요소를 생성한다.  
이는 속성 참조 체인으로 만들어진 표현식을 사용할 때 유용하다.  
배열 또는 리스트로 인덱싱하고 현 배열 혹은 리스트의 크기를 초과하는 인덱스를 지정하면 SpEL은 해당 배열 또는 리스트를 자동으로 늘릴 수 있다.  
지정된 인덱스에 요소를 추가하기 위해 SpEL은 해당 유형의 기본 생성자를 이용하여 요소를 생성하려고 한다.  
요소 유형에 기본 생성자가 없으면 null로 추가된다.  

다음 예제는 목록을 자동으로 늘리는 방법에 대한 예제이다.  

```java
class Demo {
    public List<String> list;
}

// 2가지 기능 on
// 자동 null 참조 초기화
// 자동 컬렉션 증가
SpelParserConfiguration config = new SpelParserConfiguration(true,true);

ExpressionParser parser = new SpelExpressionParser(config);

Expression expression = parser.parseExpression("list[3]");

Demo demo = new Demo();

Object o = expression.getValue(demo);

// demo.list는 이제 4개의 요소를 갖는 컬렉션이다.
// 각 요소는 빈 문자열이다.
```

## 빈 정의에서의 Expressions

빈 정의 인스턴스를 정의하기 위해 XML 기반 혹은 어노테이션 기반의 SpEL 표현식을 사용할 수 있다.  
두 경우 모두 표현식을 정의하는 구문은 `#{ <표현식 문자열> }` 이다.  

### 어노테이션 설정

`@Value` 어노테이션을 사용하여 필드, 메서드, 생성자 파라미터 등에 사용할 수 있다.  

```java
public class MovieRecommender {

    private String defaultLocale;

    private CustomerPreferenceDao customerPreferenceDao;

    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao,
            @Value("#{systemProperties['user.country']}") String defaultLocale) {
        this.customerPreferenceDao = customerPreferenceDao;
        this.defaultLocale = defaultLocale;
    }

    // ...
}
```

## Language 문서

### 리터럴 표현식

리터럴 표현식은 문자열, 숫자값, boolean값 그리고 null을 지원한다.  
문자열은 홑따옴표로 구분된다.  

```java
ExpressionParser parser = new SpelExpressionParser();

// "Hello World"로 평가
String helloWorld = (String) parser.parseExpression("'Hello World'").getValue();

double avogadrosNumber = (Double) parser.parseExpression("6.0221415E+23").getValue();

// 2147483647로 평가
int maxValue = (Integer) parser.parseExpression("0x7FFFFFFF").getValue();

boolean trueValue = (Boolean) parser.parseExpression("true").getValue();

Object nullValue = parser.parseExpression("null").getValue();
```

숫자는 음수 표기, 지수 표기, 소수점 표기를 지원한다.  
기본적으로 실수는 Double.parseDouble()로 파싱된다.  

### 속성 값, 배열, 리스트, 맵, 그리고 인덱서

속성 참조를 탐색하는 것은 쉽다.  
마침표를 통해 내부 속성 값을 조회할 수 있다.  

```java
int year = (Integer) parser.parseExpression("birthdate.year + 1900").getValue(context);

String city = (String) parser.parseExpression("placeOfBirth.city").getValue(context);
```

> 참고로 `PlaceOfBirth.city` 처럼  첫글자가 대문자여도 된다.  
또한 `getPlaceOfBirth().getCity()` 와 같이 메서드 참조 형태로도 사용할 수 있다.

배열과 리스트는 브라켓으로 표현할 수 있다.  

```java
ExpressionParser parser = new SpelExpressionParser();
EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();

// Inventions 배열

// "Induction motor"로 평가
String invention = parser.parseExpression("inventions[3]").getValue(
        context, tesla, String.class);

// Members 리스트

// "Nikola Tesla"로 평가
String name = parser.parseExpression("members[0].name").getValue(
        context, ieee, String.class);

// 리스트와 배열 탐색
// "Wireless communication"으로 평가
String invention = parser.parseExpression("members[0].inventions[6]").getValue(
        context, ieee, String.class);
```

맵에서는 특별한 문자열 키를 넣어서 표현할 수 있다.  

```java
// Officer's Dictionary

Inventor pupin = parser.parseExpression("officers['president']").getValue(
        societyContext, Inventor.class);

// "Idvor"로 평가
String city = parser.parseExpression("officers['president'].placeOfBirth.city").getValue(
        societyContext, String.class);

// 값 세팅
parser.parseExpression("officers['advisors'][0].placeOfBirth.country").setValue(
        societyContext, "Croatia");
```

### 인라인 리스트

`{}` 를 사용하여 리스트를 직접적으로 표현할 수 있다.  

```java
// 4개의 숫자를 가진 리스트로 평가
List numbers = (List) parser.parseExpression("{1,2,3,4}").getValue(context);

List listOfLists = (List) parser.parseExpression("{{'a','b'},{'x','y'}}").getValue(context);
```

`{}` 는 그 자체로 빈 리스트를 의미한다.  

### 인라인 맵

`{key:value}` 형태로 맵을 표현할 수 있다.  

```java
// 2개의 요소를 가진 맵으로 평가
Map inventorInfo = (Map) parser.parseExpression("{name:'Nikola',dob:'10-July-1856'}").getValue(context);

Map mapOfMaps = (Map) parser.parseExpression("{name:{first:'Nikola',last:'Tesla'},dob:{day:10,month:'July',year:1856}}").getValue(context);
```

`{:}` 는 그 자체로 빈 맵을 의미한다.  

### 배열 생성

친숙한 자바 문법으로 배열을 생성할 수 있다.  

```java
int[] numbers1 = (int[]) parser.parseExpression("new int[4]").getValue(context);

// 초기값을 포함한 배열
int[] numbers2 = (int[]) parser.parseExpression("new int[]{1,2,3}").getValue(context);

// 다중 배열
int[][] numbers3 = (int[][]) parser.parseExpression("new int[4][5]").getValue(context);
```

다중 배열의 경우에는 초기값을 선언할 수 없다.  

### 메서드

다음과 같이 메서드를 호출할 수도 있다.  

```java
// 문자열 리터럴, "bc"로 평가
String bc = parser.parseExpression("'abc'.substring(1, 3)").getValue(String.class);

// true로 평가
boolean isMember = parser.parseExpression("isMember('Mihajlo Pupin')").getValue(
        societyContext, Boolean.class);
```

### 연산자

관계 연산자도 지원이 된다.  

```java
// true로 평가
boolean trueValue = parser.parseExpression("2 == 2").getValue(Boolean.class);

// false로 평가
boolean falseValue = parser.parseExpression("2 < -5.0").getValue(Boolean.class);

// true로 평가
boolean trueValue = parser.parseExpression("'black' < 'block'").getValue(Boolean.class);
```

> null과의 비교 시 null은 0이 아니라 없는 값으로 비교된다.  
null은 그 어떤 값보다 작은 값으로 평가된다. (`X > null` 은 항상 true, `X < null` 은 항상 false)

SpEL은 `instanceof` 와 `matches` 도 지원한다.  

```java
// false로 평가
boolean falseValue = parser.parseExpression(
        "'xyz' instanceof T(Integer)").getValue(Boolean.class);

// true로 평가
boolean trueValue = parser.parseExpression(
        "'5.00' matches '^-?\\d+(\\.\\d{2})?$'").getValue(Boolean.class);

// false로 평가
boolean falseValue = parser.parseExpression(
        "'5.0067' matches '^-?\\d+(\\.\\d{2})?$'").getValue(Boolean.class);
```

> 원시 타입은 그 즉시 래퍼 타입으로 박싱된다.  
`1 instanceof T(int)` 는 false로 평가되고, `1 instanceof T(Integer)` 는 true로 평가된다.

각 심볼릭 연산자는 영문 표현으로도 작용한다.  

- `lt` (`<`)
- `gt` (`>`)
- `le` (`≤`)
- `ge` (`≥`)
- `eq` (`==`)
- `ne` (`≠`)
- `div` (`/`)
- `mod` (`%`)
- `not` (`!`)

논리 연산자도 지원한다.  

- `and` (`&&`)
- `or` (`!!`)
- `not` (`!`)

```java
// -- AND --

// false로 평가
boolean falseValue = parser.parseExpression("true and false").getValue(Boolean.class);

// true로 평가
String expression = "isMember('Nikola Tesla') and isMember('Mihajlo Pupin')";
boolean trueValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);

// -- OR --

// true로 평가
boolean trueValue = parser.parseExpression("true or false").getValue(Boolean.class);

// true로 평가
String expression = "isMember('Nikola Tesla') or isMember('Albert Einstein')";
boolean trueValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);

// -- NOT --

// false로 평가
boolean falseValue = parser.parseExpression("!true").getValue(Boolean.class);

// -- AND and NOT --
String expression = "isMember('Nikola Tesla') and !isMember('Mihajlo Pupin')";
boolean falseValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);
```

수식 연산도 다음과 같이 지원한다.  

```java
// 덧셈
int two = parser.parseExpression("1 + 1").getValue(Integer.class);  // 2

String testString = parser.parseExpression(
        "'test' + ' ' + 'string'").getValue(String.class);  // 'test string'

// 뺄셈
int four = parser.parseExpression("1 - -3").getValue(Integer.class);  // 4

double d = parser.parseExpression("1000.00 - 1e4").getValue(Double.class);  // -9000

// 곱셈
int six = parser.parseExpression("-2 * -3").getValue(Integer.class);  // 6

double twentyFour = parser.parseExpression("2.0 * 3e0 * 4").getValue(Double.class);  // 24.0

// 나눗셈
int minusTwo = parser.parseExpression("6 / -3").getValue(Integer.class);  // -2

double one = parser.parseExpression("8.0 / 4e0 / 2").getValue(Double.class);  // 1.0

// mod 연산
int three = parser.parseExpression("7 % 4").getValue(Integer.class);  // 3

int one = parser.parseExpression("8 / 5 % 2").getValue(Integer.class);  // 1

// 연산자 우선순위
int minusTwentyOne = parser.parseExpression("1+2-3*8").getValue(Integer.class);  // -21
```

할당 연산자는 `=` 이다.  
이는 일반적으로 `setValue()` 호출 내에서 수행되지만 `getValue()` 호출 내에서도 수행될 수 있다.  

```java
Inventor inventor = new Inventor();
EvaluationContext context = SimpleEvaluationContext.forReadWriteDataBinding().build();

parser.parseExpression("name").setValue(context, inventor, "Aleksandar Seovic");

// 대안
String aleks = parser.parseExpression(
        "name = 'Aleksandar Seovic'").getValue(context, inventor, String.class);
```

### 타입

특별히 `java.lang.Class` 의 인스턴스인 `T` 연산자를 사용할 수 있다.  
정적 메서드는 이 연산자를 사용해서도 호출할 수 있다.  

```java
Class dateClass = parser.parseExpression("T(java.util.Date)").getValue(Class.class);

Class stringClass = parser.parseExpression("T(String)").getValue(Class.class);

boolean trueValue = parser.parseExpression(
        "T(java.math.RoundingMode).CEILING < T(java.math.RoundingMode).FLOOR")
        .getValue(Boolean.class);
```

### 생성자

`new` 연산자를 사용하여 생성자를 호출할 수 있다.  

```java
Inventor einstein = p.parseExpression(
        "new org.spring.samples.spel.inventor.Inventor('Albert Einstein', 'German')")
        .getValue(Inventor.class);

// 리스트의 add() 메서드를 호출하면서 Inventor 인스턴스 생성
p.parseExpression(
        "Members.add(new org.spring.samples.spel.inventor.Inventor(
            'Albert Einstein', 'German'))").getValue(societyContext);
```
