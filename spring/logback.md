# Logback
#TIL/spring

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


































