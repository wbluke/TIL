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
