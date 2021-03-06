# gradle 정리하기
#TIL/gradle

---

## Gradle이란?

### 빌드 도구

Gradle은 기존에 사용되던 Ant나 Maven처럼 프로젝트 소스 코드의 빌드를 도와주는 빌드 도구이다.  
빠르게 인기가 늘어나는 이유는 Gradle이 `스크립트 기반`이기 때문이다.  

Gradle은 Groovy 기반이고, Groovy는 다시 JVM 기반의 언어이기 때문에 문법 구조가 자바와 비슷하다.  
또한 자바에 없는 기능인 클로저와 같은 기능을 가지고 있다.  

Gradle은 빌드 스크립트를 활용하여 다양한 기능을 사용할 수 있도록 DSL(Domain Specific Language) 문법도 제공하고 있다.  


### Gradle VS. Maven

Maven은 XML을 기반으로 빌드 환경을 구성하다 보니 프로젝트의 규모가 커질수록 해당 코드의 양도 너무 많아지는 단점이 있다.  
Gradle은 DSL을 통해 Ant, Maven의 장점을 모두 수용하면서도 **간결하고 짧은 스크립트**의 구성을 가지고 있다.  

또한 Gradle 공식 홈페이지에서 Gradle이 Maven보다 최대 100배나 빠르다고 주장하고 있는데, 그 이유를 세 가지로 이야기 하고 있다.  

- 빌드 수행 시 작업의 입출력을 추적하여 최대한 변경된 부분만을 처리해 중복 작업을 수행하지 않는 증분적인 방법(Incrementally)으로 빌드를 수행한다.
- 빌드 캐시(Build Cache)를 사용한다.
- Gradle Daemon을 사용하여 빌드 관련 정보가 메모리에서 최신의 상태를 유지하도록 프로세스를 사용한다.

---

## Gradle과 빌드

### 빌드란?

소스 코드의 컴파일, 배포, 문서화 등을 수행하는 일련의 작업을 의미한다.  
예를 들어 자바 애플리케이션 같은 경우 작성이 완료된 소스 코드를 컴파일하여 클래스 파일을 생성하고, 해당 소스 코드와 관련된 프로젝트 리소스들을 JAR 등으로 압축하여 서버에 배포하게 되는데, 이런 작업을 빌드라고 할 수 있다.  

`빌드 자동화`는 반복적으로 수행되는 빌드를 자동화 또는 스크립트 파일로 만들어 적용 대상 소스 코드에 대한 작업을 자동으로 수행할 수 있도록 하는 것이다.  


### Gradle의 빌드

Gradle의 스크립트는 3가지 종류로 나눌 수 있다.  
각 스크립트는 대응되는 Gradle 기본 내장 객체가 존재한다.  

- 초기화 스크립트 (init script) - Gradle 객체
	- `init.gradle`
	- 사용자 정보, 실행 환경, 초기 선언 값 등의 정보를 기술한다.
- 설정 스크립트 (setting script) - Settings 객체
	- `settings.gradle`
	- 이 파일의 존재 유무를 통해 해당 프로젝트가 싱글 프로젝트인지, 멀티 프로젝트인지를 판별한다. 현재 작업중인 디렉토리를 시작으로 상위 디렉토리를 탐색하면서 해당 파일을 찾다가, 결과가 없다면 싱글 프로젝트, 있다면 멀티 프로젝트로 인식한다.
- 빌드 스크립트 (build script) - Project 객체
	- `build.gradle`
	- 빌드 수행과 관련된 의존 관계 정의, 태스트 정의 등의 내용이 기술된다.

Gradle의 속성 파일은 `gradle.properties`로 명명된다.  
해당 파일이 있다면 빌드 수행 시 자동으로 참조하여 해당 파일의 내용을 확인한다.  

`buildSrc`는 Gradle이 빌드를 수행할 때 클래스 파일이나 플러그인을 저장하여 참조하는 디렉토리를 의미한다.  


### Gradle의 빌드 수행

다음과 같은 6개의 단계를 가지며, **스크립트 초기화, 프로젝트 설정, 태스크 실행**의 세 단계를 묶어 `Gradle의 생명 주기`로 정의하기도 한다.  

- 명령어 해석/수행, Gradle 실행
	- 명령어를 해석하고 옵션 및 인수를 확인한다.
	- 디렉터리, 속성 파일을 확인하고 클래스 인스턴스를 생성한다.
	- 실행 모드를 판단한다. (기본 CLI 모드, 데몬 모드 등)
- 스크립트 초기화
	- 스크립트 파일을 읽고 settings.gradle 파일의 유무를 통해 싱글 프로젝트인지 멀티 프로젝트인지를 판단한다.
	- Project 객체를 생성하고 명령어 옵션 및 인수를 설정한다.
- 프로젝트 설정
	- 프로젝트에서 참조하고 있는 각종 라이브러리들을 읽어들인다.
	- 프로젝트에서 빌드를 수행하기 위해 정의되어 있는 태스크들을 확인하여 태스크들 사이의 의존 관계를 확인하고, 확인한 의존 관계를 바탕으로 `태스크 그래프`를 생성한다. 이 그래프는 순환 구조는 아니며, 방향성이 있는 비순환 그래프이다.
- 태스크 실행
	- 태스크 그래프에서 태스크를 확인하고 빌드를 실행한다.
- 결과 출력


### Gradle의 스크립트 파일

Groovy는 스크립트 언어로 JVM에서 동작하는 JVM 언어 중 하나이고 DSL에 의한 확장성이 좋은 언어이다.  
Groovy의 가장 큰 특징은 자바와 호환된다는 것이다.  

몇 가지 Groovy의 특성을 알아보자.  

```groovy
String id = 'gradle' // 형 지정
def id = 'gradle' // 형 지정 생략 (def)
println id.toUpperCase() // 속성 참조 가능

def id = "ID : ${project.id}" // 큰 따옴표는 $로 동적으로 내용을 지정할 때 사용 ({}는 생략 가능)

def id = '''gradle
         Groovy
         script''' // 작은 따옴표, 큰 따옴표 모두 세 개 연달아 사용하여 여러 줄의 문자열 지정 가능

println('hello') // 메서드 호출 시 괄호를 생략할 수 있다. 모든 경우에 가능한 것은 아니다.
println 'hello'

def id = { closer -> println "id, $closer"} // 클로저 사용
id.call('gradle') // call() 이용한 사용
id('gradle') // 일반 메서드 호출 방식 사용

def id = ['gradle', 'Groovy'] // 리스트는 배열처럼 사용 가능
id[1] = 'script'
assert id[1] == 'script' // '==' 를 이용하여 문자열을 비교, 객체 비교 시에는 is()를 사용해야 한다.

def id = ['a': 'gradle', 'b':'Groovy'] // 맵도 대괄호로 정의 가능
assert id['a'] == 'gradle'
```


Gradle의 스크립트 파일은 두 가지 요소로 구성되어 있는데, `처리문(statement)` 영역과 `스크립트 블록(script block)` 영역이다.
Gradle의 스크립트는 여러 개의 처리문과 여러 개의 스크립트 블록으로 구성되어 있다.  

```groovy
def id = 'gradle' // 처리문

repositories { // 스크립트 블록
    mavenCentral()
}
```

주요 스크립트 블록은 다음과 같다.  

- repositories
	- 저장소 설정
- dependencies
	- 의존 관계 설정
- buildscript
	- 빌드 스크립트 클래스 패스 설정
- initscript
	- 초기화 스크립트 설정
- configurations
	- 환경 구성 설정
- allprojects
	- 서브 프로젝트 포함 전체 프로젝트 설정
- subprojects
	- 서브 프로젝트 설정
- artifacts
	- 빌드 결과에 대한 설정

Gradle의 스크립트 변수는 다음과 같다.  
지역 변수, 시스템 속성, 확장 속성은 Gradle의 모든 스크립트 파일에서 사용 가능하고, 프로젝트 속성은 빌드 스크립트에서만 사용 가능하다.  

- 지역 변수
	- 선언된 부분에서 영향력이 있다.
- 시스템 속성
	- 시스템 관련 정보를 저장하는 변수로 -D 옵션이나 --system-prop 옵션으로 시스템 변수를 설정할 수 있다.
- 확장 속성
	- `ext`라는 예약어를 사용하여 스크립트 파일에서 도메인 객체의 속성을 추가할 때 사용한다. 이 속성은 ExtraPropertiesExtension 객체와 관련이 있다.
- 프로젝트 속성
	- 프로젝트에서 사용하는 변수로 -P 옵션이나 --project-prop 옵션으로 프로젝트 속성을 지정할 수 있다.

각 속성에 동일한 변수명이 있을 경우 다음과 같은 순서로 변수를 읽다가 최종적으로는 마지막 읽은 값으로 지정된다.  

- 프로젝트 디렉토리의 gradle.properties 지정
- 홈 디렉토리의 gradle.properties 지정
- 환경 변수 지정
- 명령어 옵션 -D
- 명령어 옵션 -P

---

## Gradle 기본

### Gradle의 Task

기본 task는 다음과 같이 지정한다.  
참고로 Gradle 5.0 미만에서는 leftshift(<<) 연산자로 task를 지정했으나 해당 버전부터 leftshift 연산자는 F/O 되었다.  

```gradle
task hello {
    doLast {
        println 'Hello Gradle!'
    }
}
```

따라서 항상 doFirst, doLast 등으로 task임을 지정해 줘야 한다.  
해당 블록을 지정하지 않으면 task가 아닌 설정(Configure)으로 인식이 된다.  

```gradle
task hello {
    println 'Configure' // configure

    doLast {
        println 'Hello Gradle!' // task
    }
}
```

Gradle 라이프 사이클에서 설정은 프로젝트 설정 단계에서 수행되고 실행은 태스크 실행 단계에서 수행되기 때문에 configure 부분이 task보다 먼저 실행된다.  

그리고 다음과 같이 dependsOn으로 의존 관계를 지정할 수 있다.  

```gradle
3.times { counter ->
    task "task$counter" {
        doLast {
            println "task counter : $counter"
        }
    }
}

task1.dependsOn task0, task2

// 실행 결과
// task0은 단독 실행
// task2도 단독 실행
// task1을 실행하면 task0, task2도 같이 실행
```

주요 키워드를 정리해 보자.  

- ext
	- 특정 task 안에 확장 변수를 지정하고 다른 task에서 가져다가 쓸 수 있다.
- defaultTask 'task1' 'task2'
	- 빌드 수행 시 task 이름을 지정하지 않더라도 기본 실행되도록 할 수 있다.
- task.onlyIf
	- onlyIf 블록에 해당 task의 실행 조건을 걸어 조건에 따른 빌드를 구성할 수 있다.  
- mustRunAfter, shouldRunAfter
	- 두 task를 순서를 지정해서 실행할 수 있다.
	- 의존 관계와 다른 점은 두 task를 모두 실행하도록 명시해야 한다는 것이다.
- task1.finalizedBy task2
	- finally 와 같은 개념으로 task1 실행 중 에러가 나더라도 반드시 task2를 실행하고 종료할 수 있게 한다.








































