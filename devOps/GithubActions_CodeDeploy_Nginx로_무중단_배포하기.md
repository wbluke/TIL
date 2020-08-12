# Github Actions + CodeDeploy + Nginx 로 무중단 배포하기
#TIL/devOps

---

## 개요

안녕하세요!  
이번 글에서는 제목에서와 같이 Github Actions 와 CodeDeploy, 그리고 Nginx 를 사용하여 최소 규모의 무중단 배포를 진행하는 방법에 대해 정리해보려고 합니다.  
관련 코드는 [Github 저장소](https://github.com/wbluke/playground) 에서 확인하실 수 있습니다.  


### 전체 흐름도

CI/CD 와 같이 인프라, 배포 환경을 구축하기 위해서는 내가 만들고자 하는 전체 그림을 숙지하는 것이 가장 중요하다고 생각합니다.  
그래야 진행~~삽질~~ 중에 막혔을 때 어느 부분이 문제일지 빠르게 유추해 볼 수 있기 때문입니다.  

[사진]

이 그림이 지금부터 하나씩 만들어 볼 배포 플로우입니다.  
차근차근 진행해보도록 하겠습니다.  


### 간단한 API 를 제공하는 프로젝트

먼저 배포에 사용될 간단한 Spring Boot 프로젝트를 만들어 보겠습니다.  

> 이하 예제 코드와 캡처에 나오는 프로젝트, Controller 등의 이름은 크게 신경쓰지 않으셔도 됩니다.
> 각자 본인 프로젝트에 맞게 적용하시면 됩니다.

사용할 API는 한 두개 정도일 것이기 때문에 의존성은 `starter-web` 정도만 있으면 됩니다.  

```script
// build.gradle

dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-web'
}
```

그리고 다음과 같이 아주 간단한 Controller 를 만들어 보겠습니다. 

```java
@RestController
public class LoggingController {

    @Value("${logging-module.version}")
    private String version;

    @GetMapping("/")
    public String version() {
        return String.format("Project Version : %s", version);
    }

    @GetMapping("/health")
    public String checkHealth() {
        return "healthy";
    }

}
```

루트 ("/") 로 접속하면 현재 프로젝트의 버전을 String 으로 보여주는 API 하나와 health-check 용으로 사용될 API 하나를 만들었습니다.  
프로젝트의 버전은 application.yml 에서 관리하면서 @Value 어노테이션으로 가져오도록 하겠습니다.  

```script
// application.yml

logging-module:
  version: 0.0.1
```

> 루트("/")로 접속했을 때의 화면이나 health-check 용으로 사용할 API 는 자유롭게 만드시면 됩니다.
> 이 프로젝트의 주된 목적은 아니니까요!

아래와 같이 정말 간단한 형태의 프로젝트가 완성되었습니다!

[사진]



## Github Actions








## S3 에 빌드한 jar 업로드










## CodeDeploy








## Nginx

