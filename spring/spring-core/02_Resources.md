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

