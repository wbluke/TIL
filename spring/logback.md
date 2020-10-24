# Logback
#TIL/spring

---

## Logback

안녕하세요~ 이번 포스팅에서는 Logback의 구조와 사용 방법에 대해서 정리해보려고 합니다.  
모든 내용을 다 다룰 수는 없지만, 기본적인 구조를 인지하고 있으면 나머지는 필요할 때마다 찾아보면서 적용할 수 있습니다.  
특히 Logback은 공식 문서가 꼼꼼하게 되어있는 편이어서, 없는 것 빼고 다 있다는 느낌이 들기도 합니다. ~~영어지만요~~  


### Logback이란?

먼저 Logback이 어떤 것인지 잠깐 짚고 넘어가도록 하겠습니다.  
간단히 말해서 java.util.logging, log4j, log4j2를 잇는 **자바 로깅 프레임워크**인데요.  
위 유틸들에 비해서 좋은 성능을 가지고 있으며, 특히 Spring boot에서는 기본 로깅 모듈로 채택하고 있기 때문에 많은 분들이 알게 모르게 Spring boot를 사용하시면서 접하셨을 것입니다.  

> spring-boot-starter-web 패키지 안에 Logback이 포함돼 있습니다.  


### Logback으로 로그 관리하기

Logback으로 정확히 뭘 할 수 있는가? 에 대해서 생각해보겠습니다.  
로그라는 것은 사용하기에 따라 애플리케이션에서 일어나는 일들을 기록으로 남겨서, 실시간으로, 혹은 과거에 일어났던 사건을 역으로 트래킹할 수 있는 좋은 자료라고 할 수 있는데요.  
그렇기에 로그의 내용에 따라 **적정한 로그 레벨과 로깅 위치**를 결정하는 것은 아주 중요한 일일 것입니다.  

더불어 애플리케이션에 로그를 잘 남기는 것도 매우 중요하지만, 잘 남긴 로그를 어디서 어떤 방식으로 보관할지, 어떻게 이 로그들을 확인하고 분석할 수 있을지 고민하는 일도 상당히 중요한 일입니다.  
이를 위해 Logback 설정 파일로 다음과 같은 것들을 지정할 수 있습니다.  

- 로그를 콘솔에 출력만 할지, 파일로 남길지, 네트워크를 통해 외부로 바로 전달할지 등을 결정하기
- Spring Profile 별로 로그 설정을 다르게 가져가기
- 일정 로그 레벨 이상의 로그들만 남기기
- 로그의 형식 지정하기
- 일정 시간이 지날 때마다, 시간 별로 로그 파일을 정리하거나 로그 파일을 압축해서 보관하기

또한 이렇게 로그를 파일로 남기면, [ELK Stack](https://www.elastic.co/kr/what-is/elk-stack) 같은 데이터 모니터링 도구를 활용해서 필요한 로그를 검색하거나 분석할 수도 있습니다.  


## Logback 설정하기

### logback-spring.xml

이제 Spring boot에서 Logback 설정하는 법을 알아보도록 하겠습니다.  
먼저 build.gradle에는 다음과 같이 spring-boot-starter-web 을 지정하도록 하겠습니다.  
그리고 lombok 사용을 위한 설정도 같이 넣어주도록 하겠습니다.  

```javascript
// build.gradle

dependencies {
    implementation 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    runtimeOnly 'com.h2database:h2'

    implementation 'org.springframework.boot:spring-boot-starter-web'
}
```

그리고 'src/main/resources/' 밑에 `logback-spring-xml` 파일을 생성합니다.  
일반적으로 Logback 설정은 logback.xml로 하지만, Spring boot에서의 Logback 관리를 위해서는 logback-spring.xml로 작성해야 합니다.  

[image:545F5194-5AF8-4297-A83A-06AA5B5ED50B-366-00000A1C856B0B06/2A5A79A7-0B89-40A3-9DA2-E1CC030BB44D.png]

제가 작성할 logback-spring.xml의 내용은 다음과 같습니다.  
하나씩 살펴보도록 하겠습니다!  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <include resource="org/springframework/boot/logging/logback/console-appender.xml"/>

    <property name="LOG_PATH" value="./logs"/>

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
        </encoder>
        <file>${LOG_PATH}/logback.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/application-%d{yyyy-MM-dd-HH-mm}.%i.log</fileNamePattern>
            <maxHistory>10</maxHistory>
            <totalSizeCap>${LOG_FILE_TOTAL_SIZE_CAP:-0}</totalSizeCap>
            <cleanHistoryOnStart>${LOG_FILE_CLEAN_HISTORY_ON_START:-false}</cleanHistoryOnStart>
            <maxFileSize>${LOG_FILE_MAX_SIZE:-10MB}</maxFileSize>
        </rollingPolicy>
    </appender>

    <!-- Spring Profiles -->
    <springProfile name="local">
        <include resource="org/springframework/boot/logging/logback/base.xml"/>
    </springProfile>

    <springProfile name="develop">
        <property name="LOG_PATH" value="/home/ec2-user/logs"/>

        <root level="INFO">
            <appender-ref ref="FILE"/>
        </root>
    </springProfile>

    <springProfile name="prod">
        <property name="LOG_PATH" value="/home/ec2-user/logs"/>

        <root level="INFO">
            <appender-ref ref="FILE"/>
        </root>
    </springProfile>

</configuration>
```


### logback default xml

가장 먼저 보이는 include 태그에는, defaults.xml과 console-appender.xml을 불러오는 것을 확인하실 수 있는데요.  
`org.springframework.boot.logging.logback` 패키지에는 다음과 같은 4개의 기본적인 xml 설정파일이 존재합니다.  

[image:D020134A-8B5F-439F-8FDE-C6DBB06558F7-366-00001304BF89FEEF/BA1DD5D3-A394-4D2E-A443-260DC7C34991.png]

이 중에서 `base.xml`이 나머지 세 개의 파일을 사용하고 있는 구조인데요.  
내용은 다음과 같습니다.  

```xml
<!-- base.xml -->

<included>
	<include resource="org/springframework/boot/logging/logback/defaults.xml" />
	<property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/spring.log}"/>
	<include resource="org/springframework/boot/logging/logback/console-appender.xml" />
	<include resource="org/springframework/boot/logging/logback/file-appender.xml" />
	<root level="INFO">
		<appender-ref ref="CONSOLE" />
		<appender-ref ref="FILE" />
	</root>
</included>
```


```xml
<!-- defaults.xml -->

<included>
	<conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
	<conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
	<conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />
	<property name="CONSOLE_LOG_PATTERN" value="${CONSOLE_LOG_PATTERN:-%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
	<property name="FILE_LOG_PATTERN" value="${FILE_LOG_PATTERN:-%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}} ${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%t] %-40.40logger{39} : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>

  <!-- 생략 -->
</included>
```


### springProfile

하단부에 보이는 springProfile은 application.yml에서 설정한 Profile 별로 로그 생성 전략을 다르게 설정할 수 있는 태그입니다.  

local profile에서는 `base.xml`이라는 파일을 include했고, develop, prod profile에서는 로그가 쌓일 기본 디렉토리와 로그 레벨을 설정했습니다.  



### Appender의 상속 구조




### Appender의 RollingPolicy


---

- Logback은 무엇인가?  
	- java.util.logging, log4j, log4j2에 이은 자바 로깅 프레임워크다.
	- 위 유틸들에 비해 좋은 성능을 가지고 있으며, Spring boot에서는 기본 로깅 모듈로 채택하고 있다. (spring-boot-starter-web)

- Appender는 무엇인가?  
	- Logback이 logging event를 발행하도록 위임하는 객체이다.  
	- 실제 `Appender`는 간단한 인터페이스이고, 알려진 구현체는 아래와 같은 구조를 가지고 있다.
	- name 기반의 getter, setter와 이벤트를 수집하는 `void doAppend(E event);` 메소드를 가지고 있다.


[image:1357571C-4D06-4301-BD61-7BBC5B51C29F-395-00001C63A41A9C0B/D5EAB5E9-8D72-4B19-A0DC-574B8B1B5E6B.png]

![](./images/logback01.png)

- AppenderBase
	- Appender 를 구현하고 있는 추상클래스
	- Appender의 doAppend(E event) 메소드를 synchronized 로 구현하고 있다.
		- 이 때문에 멀티스레드 환경에서 같은 Appender를 사용하더라도 안전하다.

- UnsynchronizedAppenderBase
	- 하지만 synchronization이 늘 적절하지는 않기 때문에 logback은 AppnederBase와 유사하지만 synchronized 키워드가 없는 구현체를 제공한다.

- OutputStreamAppender
	- Event를 OutputStream으로 발행하는 객체
	- 사용자가 직접 생성할 일은 없다.
	- ConsoleAppender와 FileAppender의 부모 클래스가 된다.
	- `encoder` property를 갖고 있다.
		- 로그를 어떤 형식으로 남길 것인지를 결정한다.

- ConsoleAppender
	- PrintStream을 사용하여 Console에 로그를 남기는 Appender

- FileAppender
	- 로그를 file로 남기는 Appender
	- append
		- 기본값 true. true면 이미 file이 존재하는 경우 이어서 쓰고, false면 기존 file에 덮어쓴다.
	- file
		- 로그를 남길 file의 이름이다. 해당 파일이 없으면 새로 생성한다.

- RollingFileAppender
	- FileAppender를 상속한다.
	- 특정 조건이 되면, 로그를 쌓던 타겟 파일을 그 다음 파일로 변경한다. (rollover)
	- 두 가지 중요한 속성이 있다.
		- `RollingPolicy`
			- rollover 를 어떻게 실행할지를 정의한다.
		- `TriggeringPolicy`
			- 언제 rollover를 발생시킬지를 정의한다.

RollingPolicy는 인터페이스이고, 이를 구현하는 여러 구현체들이 또 존재한다.  



- TimeBasedRollingPolicy
	- 이 자체만으로도 TriggeringPolicy도 구현하고 있다.
	- fileNamePattern
		- rollover된 파일 이름을 정의한다.
		- SimpleDateFormat 패턴을 기준으로 한다.
		- **rollover의 기간은 fileNamePattern에 따라 정해진다.**
			- `%d{yyyy-MM-dd}.log`
				- 하루치씩 쌓인다.
			- `%d{yyyy-MM-dd_HH}.log`
				- 매 시간 단위로 쌓인다.
		- 부모인 FileAppender의 File 속성과 함께 쓰는 경우
			- File 속성은 현재 쌓이는 로그의 타겟 파일 이름이다.
			- File 속성을 생략하면 FileNamePattern에 의해 현재 로그가 쌓이는 파일의 이름이 정해진다.
	- maxHistory
		- 최대로 보관하는 파일의 개수를 정한다.
		- rollover 단위가 월 단위라면, 6개의 파일(6개월 치)을 보관하고 오래된 순으로 삭제한다.
	- totalSizeCap
		- 아카이빙한 로그 파일의 최대 사이즈를 정의한다.
		- maxHistory와 함께 정의되어야 하며, maxHistory가 더 우선적인 조건이 된다.
	- cleanHistoryOnStart
		- true값이면 appender가 시작할 때 기존 아카이빙된 파일들을 삭제한다. 기본값은 false다.

- SizeAndTimeBasedRollingPolicy
	- TimeBasedRollingPolicy + 로그 파일의 총 사이즈를 제한하고 싶을 때 사용한다.
	- maxFileSize 속성으로 정의한다.


- TriggeringPolicy 는 더 간단한데, 언제 RollingPolicy를 트리거시킬지를 정한다.
	- SizeBasedTriggeringPolicy는 maxFileSize 속성으로 트리거를 정의한다.


































