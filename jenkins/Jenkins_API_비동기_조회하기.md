## Intro

평화로운 2020년 9월의 어느 날...  

데일리 미팅을 마치고 일감을 정리하던 저에게 한 가지 요청이 들어왔습니다.  

"우빈님 여기 로직이 오래 걸리면 90초 넘게 걸리고 있는데 한번 개선할 수 있을지 확인 부탁드려요."  

'읭 아니 대체 어떤 레거시길래 90초씩이나 걸리는거야'  

라고 생각하며 코드를 열어서 확인했는데요.  

[사진]  

범인은 다섯 달 전의 저였습니다.  

개발자들에게는 흔히 있는 일이라고 하는데... 저만 겪고 있는 건 아니죠?  

오늘 포스팅에서는 위 레거시를 생산하게 된 배경과, 그 해결과정을 정리해서 공유해보려고 합니다.  

크게 어려운 내용은 아니니 해결해가는 과정 자체에 포인트를 두고 가볍게 읽어주시면 감사하겠습니다 😉  

## 5개월 전...

입사 후 [신규입사자 온보딩 과정](https://woowabros.github.io/experience/2020/03/02/pilot-project-wbluke.html)을 마무리하고, 팀에서 가장 먼저 받은 첫 번째 과제는 정산시스템 어드민(관리자 페이지)의 최초 진입 화면인 `대시보드`를 만드는 과제였습니다.  

당시 정산 어드민에 로그인한 후 보이는 첫 화면은 좌측 메뉴 바를 제외하고는 특별하게 제공되는 것이 없는 빈 화면이었는데요.  

관리자 페이지를 사용하시는 기획/운영자 분이 수시로 확인하면 좋을 각종 실시간 지표들을 화면에 구축하는 것이 바로 해당 과제의 내용이었습니다.  

### 배치 모니터링

요구사항에 있는 많은 지표들 중 하나는, 정산시스템의 허리를 담당하고 있는 수십 개의 배치를 개발자가 아닌 운영자 분이 모니터링할 수 있도록 하는 `배치 모니터링` 지표였습니다. 

> 아래에 이어서 나오는 배치 모니터링 분석에 대한 내용은 당시에 [태현님](https://woowabros.github.io/experience/2020/10/08/excel-download.html)이 많은 도움을 주셨습니다 🙏

배치 모니터링 지표에서는 그날 수행될 예정인 배치가 어떤 것인지 파악할 수 있어야 했고, 해당 배치의 실시간 상태를 확인할 수 있어야 했습니다.  

또한 각 배치의 활성화(on/off) 상태와 더불어 수행 시작/종료 시간도 트래킹할 수 있어야 했습니다.  

즉, 정리하자면 다음과 같은 정보들을 어디선가 얻어오거나 필요에 따라 새롭게 정의해야 했습니다.  

- 배치의 가장 최근 작업 상태 (수행중/성공/실패/수행예정)
- 활성화 여부
- 작업 시작시간/종료시간
- 작업 주기 (배치가 수행될 예정인지를 판별하기 위함)
    - 운영자가 매일/영업일/매월 1일/수시 등으로 사전에 정의하여 지정

저희 팀은 배치 관리 도구로 젠킨스(Jenkins)를 사용하고 있는데요.  

[사진]  

정산시스템에서 이 젠킨스를 통해 관리하고 있는 배치의 형태를 파악해보니 다음과 같이 세 종류로 구분할 수 있었습니다. (이름은 임의로 붙여봤습니다)  

[사진]  

- Simple Job
    - 가장 단순한 형태의 배치로, Spring Batch와 Jenkins Job이 1:1인 경우
- Trigger Job
    - 하나의 주요한 배치 A가 있고, A 배치에게 다양한 파라미터를 제공하며 수행 요청만 하고 자기자신은 종료되는 배치
    - 배치 A와 1:N 관계
    - 예를 들어, 배치 A에 1번 파라미터를 제공하는 배치, 배치 A에 2번 파라미터를 제공하는 배치 등
- Pipeline Job
    - 여러 개의 배치 Job을 순차적으로 실행할 수 있는 파이프라인 배치
    - 파이프라인 Script로 하위 배치 Job를 단계별로 지정해서 실행

이렇게 구해야 하는 배치의 정보와, 현재 배치 시스템의 상황을 파악하고 나니 다음과 같은 고민이 있었습니다.  

**💡 수십 개의 배치가 수행된 상태를 실시간으로 조회하기 위해 어떤 방법을 선택해야 하는가?**  

크게는 두 가지 방법을 놓고 고민했는데요.  

바로 `Spring Batch Metadata Tables`와 `젠킨스 API` 였습니다.  

### Spring Batch Metadata Tables

[사진]  

[Spring Batch Metadata Tables(Schema)](https://docs.spring.io/spring-batch/docs/current/reference/html/schema-appendix.html)는 Spring Batch로 수행된 모든 Job에 대한 정보를 가지고 있는 메타데이터 테이블입니다.  

위 요구사항과 관련한 정보들을 가져오기 위해서는, 그중에서도 BATCH_JOB_INSTANCE 테이블과 BATCH_JOB_EXECUTION 테이블, 그리고 BATCH_JOB_EXECUTION_PARAMS 테이블을 참고해야 할 것으로 보였습니다.  

Spring Batch를 사용하시는 분들은 대부분 아시겠지만, 한번 간단하게 정리해보자면 다음과 같습니다.  

- BATCH_JOB_INSTANCE
    - Job Parameter에 따른 배치 수행 정보를 기록
    - 동일 파라미터로는 데이터가 발생하지 않음 (배치 수행 불가)
- BATCH_JOB_EXECUTION
    - BATCH_JOB_INSTANCE의 자식 테이블로, 모든 성공/실패 내역을 가지고 있음
- BATCH_JOB_EXECUTION_PARAMS
    - 실제 수행되었던 배치의 구체적인 모든 파라미터가 기록되는 테이블

더 상세한 내용은 [jojoldu님의 블로그](https://jojoldu.tistory.com/326)를 참고해보시면 좋습니다.  

Spring Batch Metadata Tables와 위 요구사항을 매칭해보니, 다음과 같은 문제점들이 보였습니다.  

- 파이프라인 Job의 수행 시작시간/종료시간을 구하는 경우, 파이프라인 하위 배치들의 순서 및 첫 수행 배치와 마지막 수행 배치 시간을 알아와야 하는데, 필요 이상의 테이블 조인 및 조회가 발생하고, 추가적인 관리 포인트가 생길 수 있다.
- Jenkins Job의 활성화 여부는 Jenkins API로만 얻어올 수 있다.
    - (아래 캡쳐 참조) 배치 활성화 on/off 는 `빌드 안함` 체크로 관리하고 있다.

[사진]  

### Jenkins API

다음으로는 Jenkins의 API에 대해 고민했는데요.  

[Jenkins API wiki](https://www.jenkins.io/doc/book/using/remote-access-api/)가 있긴 하지만, 어떤 API가 있는지 정확하게 정리해주지는 않고 있습니다.  

하지만 우리의 요구사항에 대한 정보를 얻어오기 위해서는 다음 2개의 API면 충분합니다.  

첫 번째로 Job에 대한 정보를 받아오는 API 입니다.  

`${Jenkins_URL}/job/${job_name}/api/json` 으로 요청할 수 있습니다. (Jenkins job 화면에서 `/api/json` 을 붙여서 확인해보실 수 있습니다.)  

주요한 필드만 기술해보면 아래와 같습니다. (테스트 환경의 IP 정보는 JENKINS_URL로 표시했습니다.)  

```json
{
  ...

  "description": "",
  "displayName": "test-01",
  "fullDisplayName": "test-01",
  "fullName": "test-01",
  "name": "test-01",
  "url": "http://JENKINS_URL/job/test-01/",
  "buildable": true,
  "builds": [
    {
      "_class": "hudson.model.FreeStyleBuild",
      "number": 3,
      "url": "http://JENKINS_URL/job/test-01/3/"
    },
    {
      "_class": "hudson.model.FreeStyleBuild",
      "number": 2,
      "url": "http://JENKINS_URL/job/test-01/2/"
    },
    {
      "_class": "hudson.model.FreeStyleBuild",
      "number": 1,
      "url": "http://JENKINS_URL/job/test-01/1/"
    }
  ],
  "color": "blue",
  "firstBuild": {
    "_class": "hudson.model.FreeStyleBuild",
    "number": 1,
    "url": "http://JENKINS_URL/job/test-01/1/"
  },
  "inQueue": false,
  "keepDependencies": false,
  "lastBuild": {
    "_class": "hudson.model.FreeStyleBuild",
    "number": 3,
    "url": "http://JENKINS_URL/job/test-01/3/"
  },
  "lastCompletedBuild": {
    "_class": "hudson.model.FreeStyleBuild",
    "number": 3,
    "url": "http://JENKINS_URL/job/test-01/3/"
  },
  "lastFailedBuild": {
    "_class": "hudson.model.FreeStyleBuild",
    "number": 2,
    "url": "http://JENKINS_URL/job/test-01/2/"
  },
  "lastStableBuild": {
    "_class": "hudson.model.FreeStyleBuild",
    "number": 3,
    "url": "http://JENKINS_URL/job/test-01/3/"
  },
  "lastSuccessfulBuild": {
    "_class": "hudson.model.FreeStyleBuild",
    "number": 3,
    "url": "http://JENKINS_URL/job/test-01/3/"
  },
  "lastUnstableBuild": null,
  "lastUnsuccessfulBuild": {
    "_class": "hudson.model.FreeStyleBuild",
    "number": 2,
    "url": "http://JENKINS_URL/job/test-01/2/"
  },
  "nextBuildNumber": 4,
  "downstreamProjects": [
    
  ],
  "upstreamProjects": [
    
  ]
}
```

Job API에서 눈여겨 볼 정보들은 다음과 같습니다.  

- name
    - Job 이름
- buildable
    - 활성화 여부 (`빌드 안함` 체크 여부)
- lastBuild, lastCompletedBuild, lastFailedBuild
    - 최신 빌드 url, 최신 성공 빌드 url, 최신 실패 빌드 url

두 번째로는 Build에 대한 정보를 받아오는 API 입니다.  

`${Jenkins_URL}/job/${job_name}/${build_id}/api/json` 으로 요청할 수 있습니다. (상세 Build 화면에서 `/api/json` 을 붙여서 확인해보실 수 있습니다.)  

마찬가지로 주요한 필드만 기술해보면 아래와 같습니다.  

```json
{
  "building": false,
  "displayName": "#3",
  "duration": 34,
  "estimatedDuration": 116,
  "fullDisplayName": "test-01 #3",
  "id": "3",
  "keepLog": false,
  "number": 3,
  "result": "SUCCESS",
  "timestamp": 1616924008157,
  "url": "http://JENKINS_URL/job/test-01/3/",
}
```

Build API에서 눈여겨 볼 정보는 다음과 같습니다.  

- building
    - 현재 수행중인지 여부
- duration
    - 수행된 시간
- result
    - 빌드 결과
- timestamp
    - 수행 시작 시간
    - timestamp에 duration을 더해서 환산하면 수행 종료 시간이 됩니다.

위 두 API에서 가장 중요한 포인트는, 하나의 API만 가지고는 우리가 필요한 모든 정보를 얻을 수 없다는 것입니다.  

즉, Job API에서는 Build에 대한 상세 정보를 알려주지 않고, Build API에서는 Job에 대한 상세 정보(활성화 여부 등)를 알려주지 않습니다.  

[사진]  

(위 2개 API를 제외한 다른 API가 궁금하시면 Jenkins API wrapping 라이브러리인 [Jenkins-rest](https://github.com/cdancy/jenkins-rest)를 참고해보셔도 좋습니다!)  

### 해보자, 조회!

## 비동기로 로직 개선

## 1차 배포 후

## 2차 배포 후

