# CompletableFuture 톺아보기
#TIL/java

---

## CompletableFuture

안녕하세요! 이번 포스팅에서는 학습 테스트를 통해 CompletableFuture를 알아보는 시간을 가져보려고 합니다.
모든 코드는 [GitHub](https://github.com/wbluke/playground)에 있으니 참고하시면 됩니다.  

CompletableFuture는 Java에서 대표적으로 비동기 요청을 처리할 때 사용하는 객체인데요.  
사용하기에 따라 Async-Blocking, Async-Non-Blocking 하게 사용할 수 있습니다.  
제공하는 기능이 꽤 많아서 이 글에서 전부 다루지는 못하지만, 몇 가지 주요 기능을 알고 나면 나머지는 필요에 따라 적용해볼 수 있다고 생각합니다.  

> 동기 / 비동기, Blocking / Non-Blocking 에 대해 감이 잘 안오신다면 [이전 포스팅](https://wbluke.tistory.com/49)을 참조해 주세요 :)

### 학습 테스트를 위한 환경 구성

먼저 학습 테스트를 위해 환경을 구성하겠습니다.  
build.gradle은 다음과 같습니다.  

```js
// build.gradle

dependencies {
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    runtimeOnly 'com.h2database:h2'

    implementation 'org.springframework.boot:spring-boot-starter-web'

    testImplementation('org.springframework.boot:spring-boot-starter-test') {
        exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
    }
    testCompileOnly 'org.projectlombok:lombok'
}
```

특별한 것은 없고 맨 마지막 줄에 테스트 패키지에 롬복 의존성을 준 부분만 염두에 두시면 되는데요.  
테스트에서 `@Slf4j`로 log를 사용해 비동기 동작을 확인하기 위함입니다.  

다음으로는 테스트 클래스를 하나 생성해서, 다음과 같이 비동기 상황을 위한 private 메소드를 만들어 보겠습니다.  

```java
@Slf4j
public class CompletableFutureTest {

    private String sayMessage(String message) {
        sleepOneSecond();

        String saidMessage = "Say " + message;
        log.info("Said Message = {}", saidMessage);

        return saidMessage;
    }

    private void sleepOneSecond() {
        try {
            log.info("start to sleep 1 second.");
            Thread.sleep(1000);
            log.info("end to sleep 1 second.");
        } catch (InterruptedException e) {
            throw new IllegalStateException();
        }
    }

}
```

파라미터로 문자열 메시지가 들어오면, 1초 쉰 후에 "Say "를 붙여서 return하는 간단한 형태의 메소드입니다.  
이 때 정확한 동작 타이밍을 알기 위해 sleep 메소드 앞뒤로 log를 찍고, "Say "를 붙인 이후에도 log를 찍도록 하겠습니다.  

이제부터 작성할 모든 테스트는 이 기능을 기반으로 진행됩니다.  

### supplyAsync / runAsync / join

가장 먼저 비동기 동작의 기본이 되는 메소드를 알아보겠습니다.  
supplyAsync()와 runAsync(), 그리고 join()인데요!
테스트 코드를 먼저 보겠습니다.  

```java
@Test
void supplyAsync() {
    /* given */
    String message = "Hello";
    CompletableFuture<String> messageFuture = CompletableFuture.supplyAsync(() -> sayMessage(message));

    /* when */
    String result = messageFuture.join();

    /* then */
    log.info("result = {}", result);
    assertThat(result).isEqualTo("say Hello");
}
```

supplyAsync()는 파라미터로 Supplier 인터페이스를 받아서, 반환값이 존재하는 메소드입니다.  
비동기 상황에서의 작업을 콜백 함수로 넘기고, 작업 수행 여부와 관계 없이 CompletableFuture 객체로 다음 로직을 이어나갈 수 있습니다.  

when 절에서 쓰인 join()은 해당 CompletableFuture가 끝나기를 기다리는 Blocking 메소드라고 할 수 있습니다.
콜백으로 넘긴 작업이 끝나고 난 이후의 결과값을 반환하게 됩니다.  

> get()도 join()과 같은 역할을 하는데요.
> get() 메소드는 Checked Exception을 던지기 때문에 try-catch 구문이나 throws 처리를 해야 합니다.
> 따라서 Unchecked Exception 을 던지는 join()을 쓰는 것이 좋습니다.

사실 join()이 없고 supplyAsync()만으로도 CompletableFuture는 비동기적으로 "say Hello"를 만들어내는 작업을 수행합니다.
하지만 클라이언트 역할을 하고 있는 테스트 코드에서 해당 작업이 수행되었는지를 확인할 수가 없기 때문에, Blocking을 걸어 then절에서 검증 작업을 수행하도록 하겠습니다.  

[사진]

실행하면서 로그를 보시면 sleep 메소드가 1초 동안 수행되고 원하는 결과가 잘 나왔음을 볼 수 있습니다.

이어서 runAsync()를 볼까요?

```java
@Test
void runAsync() {
    /* given */
    String message = "Hello";
    CompletableFuture<Void> messageFuture = CompletableFuture.runAsync(() -> sayMessage(message));

    /* when */ /* then */
    messageFuture.join();
}
```

runAsync()는 파라미터로 Runnable 인터페이스를 받아서, 반환값이 존재하지 않습니다.  
따라서 반환값이 굳이 필요 없는 작업을 비동기로 처리할 때에 유용한 방법입니다.  

supplyAsync()에서와 마찬가지로 join()을 사용하여 Blocking을 걸고, 로그로 동작을 확인해보겠습니다.
잘 동작하는 것을 확인할 수 있습니다.


















