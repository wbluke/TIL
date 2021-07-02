## 들어가며

자바 표준 `java.net.URL` 클래스와 표준 URL 핸들러들은 불행히도, 낮은 레벨의 리소스에 접근하기에는 충분하지 않다.  
예를 들어, 클래스 경로나 ServletContext에 관련된 리소스에 접근하는 표준화된 URL 구현체가 없다.  
특별한 URL 접두사를 위한 핸들러를 등록할 수 있지만, 일반적으로 꽤 복잡하며, URL 인터페이스는 리소스가 실제로 존재하는지를 체크하는 기능 등이 부족하다.  

## Resource 인터페이스

`org.springframework.core.io` 패키지에 있는 `Resource` 인터페이스는 낮은 레벨의 리소스에 접근하기 위한 유능한 인터페이스이다.  
Resource 인터페이스는 다음과 같다.  

```java
public interface Resource extends InputStreamSource {

    boolean exists();

    boolean isReadable();

    boolean isOpen();

    boolean isFile();

    URL getURL() throws IOException;

    URI getURI() throws IOException;

    File getFile() throws IOException;

    ReadableByteChannel readableChannel() throws IOException;

    long contentLength() throws IOException;

    long lastModified() throws IOException;

    Resource createRelative(String relativePath) throws IOException;

    String getFilename();

    String getDescription();
}
```

살펴볼 메서드는 다음과 같다.  

- `getInputStream()` (InputStreamSource에 있다.)
    - 리소스를 읽어서 InputStream으로 반환해준다.
- `exists()`
    - 리소스가 물리적으로 존재하는지를 확인한다.
- `isOpen()`
    - 리소스가 stream으로 핸들링할 수 있는지를 나타낸다.

다른 메서드들은 리소스를 표현하는 URL이나 File 객체를 제공한다.  
Resource의 어떤 구현체들은 `WritableResource` 인터페이스도 구현해서 리소스에 쓰기 작업을 하기도 한다.  

스프링은 자체적으로 Resource가 필요한 경우 메서드 시그니처의 인자 타입 등으로 광범위하게 사용한다.  

## 내장된 Resource 구현체

### UrlResource

`UrlResource` 는 `java.net.URL` 을 포장하며, 파일, HTTPS 타겟, FTP 타겟 등 URL로 접근할 수 있는 어떤 객체든 사용 가능하다.  

### ClassPathResource

### FileSystemResouce

### PathResource

### ServletContextResource

### InputStreamResource

### ByteArrayResource

## ResourceLoader 인터페이스

```java
public interface ResourceLoader {

    Resource getResource(String location);

    ClassLoader getClassLoader();
}
```

## ResourcePatternResolver 인터페이스

## ResourceLoaderAware 인터페이스

ResourceLoaderAware 인터페이스는 ResourceLoader를 제공받을 것으로 예상되는 구성 요소를 식별하는 콜백 인터페이스이다.  

```java
public interface ResourceLoaderAware {

    void setResourceLoader(ResourceLoader resourceLoader);
}
```

어떤 클래스가 ResourceLoaderAware를 구현하고 애플리케이션 컨텍스트에서 관리되고 있다면, 애플리케이션 컨텍스트에 의해 ResourceLoaderAware로 인식된다.  
그런 다음 애플리케이션 컨텍스트는 setResourceLoader를 호출하여 자신을 인수로 제공한다.  
(스프링의 모든 애플리케이션 컨텍스트는 ResourceLoader 인터페이스를 구현한다.)  

ApplicationContext가 ResourceLoader이기 때문에, 빈은 또한 ApplicationContextAware 인터페이스를 구현하고 제공되는 애플리케이션 컨텍스트를 리소스 로드를 위해 사용할 수도 있다.  
하지만 일반적으로 필요한 경우 특수한 ResourceLoader 인터페이스를 사용하는 것이 좋다.  
코드가 리소스 로딩 인터페이스와만 관련이 있도록 하고, 전체 ApplicationContext 인터페이스와는 관련이 없도록 하는 것이 좋기 때문이다.  

컴포넌트에서 ResourceLoaderAware 인터페이스를 구현하는 대신 ResourceLoader를 오토와이어링할 수도 있다.  
기존 방식인 생성자나 byType 오토와이어링은 각각 생성자 인자나 setter 메서드를 통해 ResourceLoader를 제공할 수 있고, 더 많은 유연성을 원한다면 @Autowired를 사용한 어노테이션 기반 오토와이어링을 사용할 수 있다.  

## 의존성으로서의 리소스

빈 자체가 일종의 동적인 프로세스를 통해 리소스 경로를 결정하고 제공하려는 경우 빈이 ResourceLoader 또는 ResourcePatternResolver 인터페이스를 사용하여 리소르를 로드하는 것이 합리적일 수 있다.  
예를 들어 특정 리소스가 사용자의 역할에 따라 달라지는 템플릿 등이 있을 수 있다.  
리소스가 정적인 경우는 ResourceLoader 인터페이스 대신 빈이 필요한 Resource 속성을 노출하도록 해서 사용하는 것이 좋다.  

```java
package example;

public class MyBean {

    private Resource template;

    public setTemplate(Resource template) {
        this.template = template;
    }

    // ...
}
```

```html
<bean id="myBean" class="example.MyBean">
    <property name="template" value="some/resource/path/myTemplate.txt"/>
</bean>
```

리소스 경로에는 접두사가 없다.  
결과적으로 애플리케이션 컨텍스트 자체가 ResourceLoader로 사용되기 때문에 리소스는 컨텍스트 유형에 따라 ClassPathResource, FileSystemResource 또는 ServletContextResource를 통해 로드된다.  

특정 리소스 유형을 강제로 사용해야하는 경우에는 다음과 같이 클래스패스 혹은 파일 등의 접두사를 사용할 수 있다.  

```html
<property name="template" value="classpath:some/resource/path/myTemplate.txt">
```

```html
<property name="template" value="file:///some/resource/path/myTemplate.txt"/>
```

## 애플리케이션 컨텍스트와 리소스 경로

### 애플리케이션 컨텍스트 구성

애플리케이션 컨텍스트의 생성자는 문자열이나 문자열 배열로 XML 파일 같은 경로를 받아 컨텍스트를 정의한다.  
만약 경로에 접두사가 없다면, 해당 경로에서 특별한 Resource 타입이 만들어지고 이는 애플리케이션 컨텍스트의 구현체에 따라 달라진다.  

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("conf/appContext.xml");
```

위 예제에서 빈 정의는 클래스패스에 따라 로드되는데, ClassPathResource가 사용되었기 때문이다.  

```java
ApplicationContext ctx = new FileSystemXmlApplicationContext("conf/appContext.xml");
```

반면, 위 예제는 파일 시스템을 통해 빈 정의가 로드된다.  

만약 특별한 `classpath` 접두사나 표준 URL 접두사가 사용된다면, 다음과 같이 Resource의 기본 타입을 오버라이딩한다.  

```java
ApplicationContext ctx = new FileSystemXmlApplicationContext("classpath:conf/appContext.xml");
```

FileSystemXmlApplicationContext는 클래스패스에 의해 빈 정의를 로드한다.  
하지만 이는 여전히 FileSystemXmlApplicationContext이기 때문에 나중에 ResourceLoader 등으로 사용되는 경우 접두사가 없는 패스는 파일 시스템 기반으로 다뤄질 것이다.  

### FileSystemResource 주의 사항

FileSystemApplicationContext에 연결되지 않은 FileSystemResource (즉, FileSystemApplicationContext가 ResourceLoader가 아닌 경우)는 절대 경로와 상대 경로를 모두 잘 처리한다.  

하지만 이전 버전과의 호환성 이유로 인해 FileSystemApplicationContext가 ResourceLoader일 때는 절대 경로, 상대 경로에 상관 없이 모든 위치 경로를 상대 경로로 처리한다.  
즉, 다음 두 경우가 같은 방식으로 동작한다.  

```java
ApplicationContext ctx = new FileSystemXmlApplicationContext("conf/context.xml");
```

```java
ApplicationContext ctx = new FileSystemXmlApplicationContext("/conf/context.xml");
```

아래 두 예제도 마찬가지로 같은 의미다.  

```java
FileSystemXmlApplicationContext ctx = ...;
ctx.getResource("some/resource/path/myTemplate.txt");
```

```java
FileSystemXmlApplicationContext ctx = ...;
ctx.getResource("/some/resource/path/myTemplate.txt");
```

실제로 파일 절대 경로가 필요한 경우는 `file:` 접두사를 사용하여 다음과 같이 UrlResource를 사용하도록 해야 한다.  

```java
// context 타입에 상관 없이, Resource는 항상 UrlResource
ctx.getResource("file:///some/resource/path/myTemplate.txt");
```

```java
// UrlResource를 통해 정의를 로드하도록 FileSystemXmlApplicationContext 강제
ApplicationContext ctx = new FileSystemXmlApplicationContext("file:///conf/context.xml");
```
