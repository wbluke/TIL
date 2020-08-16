# Github Actions + CodeDeploy + Nginx 로 무중단 배포하기
#TIL/devOps

---

## 개요

안녕하세요!  
이번 시리즈에서는 제목에서와 같이 Github Actions 와 CodeDeploy, 그리고 Nginx 를 사용하여 최소 규모의 무중단 배포를 진행하는 방법에 대해 정리해보려고 합니다.  
관련 코드는 [Github 저장소](https://github.com/wbluke/playground) 에서 확인하실 수 있습니다.  


### 전체 흐름도

CI/CD 와 같이 인프라, 배포 환경을 구축하기 위해서는 내가 만들고자 하는 전체 그림을 숙지하는 것이 가장 중요하다고 생각합니다.  
그래야 진행~~삽질~~ 중에 막혔을 때 어느 부분이 문제일지 빠르게 유추해 볼 수 있기 때문입니다.  

[image:BCFA5E37-5520-4B6B-93F9-FD0A3854CDFA-371-0000024A9FAECF19/72567102-91C5-479E-AD3E-B5153F569409.png]

이 그림이 지금부터 하나씩 만들어 볼 배포 플로우입니다.  
차근차근 진행해보도록 하겠습니다.  

## 예제 프로젝트

### 간단한 API 를 제공하는 프로젝트

먼저 배포에 사용될 간단한 Spring Boot 프로젝트를 만들어 보겠습니다.  

> 이하 예제 코드와 캡처에 나오는 프로젝트, Controller 등의 이름은 크게 신경쓰지 않으셔도 됩니다.
> 각자 본인 프로젝트에 맞게 적용하시면 됩니다.

사용할 API는 한 두개 정도일 것이기 때문에 기본적인 설정 외에 의존성은 `starter-web` 정도만 있으면 됩니다.  

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

[image:358AC1AC-80C5-4449-87DF-7C5A0C15BAE2-379-00000C7749B306D3/78D41EE6-DC54-4154-8E71-61CA5A7C5B4E.png]



## Github Actions

### 소개

가장 먼저 구축해 볼 것은 [GitHub Actions](https://github.com/features/actions) 입니다.  

빌드와 배포를 유발하는 방법에는 정말 여러가지가 있을 수 있는데요, 작년 2019년에 갓 나온 Github Actions 는 손쉽게 개인 Github 코드 저장소에서 빌드/배포 환경을 구축할 수 있는 도구입니다.  
지금 당장 사용할 일이 없더라도 한번쯤은 사용해보고 배워두면 깃헙 저장소를 계속 이용하는 이상 정말 쓸모가 많을 것 같다는 생각이 드네요.  
심지어 public 저장소에서는 무료! 로 지원합니다.  

기본적으로 yml 파일로 스크립트를 짜기 때문에 해당 파일 형식에 익숙하시다면 금방 적응하실 수 있습니다.  

### workflow 생성하기

먼저 저장소의 Actions 에서 `set up a workflow yourself` 로 새 작업(이하 workflow)을 만들어 봅시다.  

[image:C4A58964-C301-4D20-8529-FFE35D479D24-379-0000208A72862576/4E031957-0341-4C2D-9B78-74424D4BA4D0.png]

새 workflow 를 생성하면 좌측에는 yml 스크립트를 입력할 수 있는 부분이 있고, 우측에는 미리 만들어져 있는 스크립트를 필요에 따라 추가할 수 있는 `Marketplace` 와 workflow 에 대한 설명을 볼 수 있는 `Documentation` 이 있습니다.  
Documentation 에는 비교적 쉽게 workflow 의 소개가 되어 있어서 한 번 참고해보시면 좋을 것 같습니다.  

그리고 눈치 채신 분들도 계시겠지만 workflow yml 파일은 상단에서 볼 수 있다시피 `프로젝트/.github/workflows/` 위치에 생성됩니다.  

[image:CEBF4E53-5104-4877-806C-17B8C8E8D57C-379-000021F98D7FE1FB/6032CF47-1A73-45EE-A4D4-D4E1259B76EC.png]

좌측에 예시로 나와있는 설명도 간단하게 훑어보셨다면, 다음과 같이 스크립트를 작성해 보겠습니다.  

```yml
# logging-deploy.yml

name: logging-system

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v2
      
      - name: Set up JDK 1.8
        uses: actions/setup-java@v1
        with:
          java-version: 1.8
      
      - name: Grant execute permission for gradlew
        run: chmod +x gradlew
        shell: bash

      - name: Build with Gradle
        run: ./gradlew build
        shell: bash
      
# 다음 단계에서 스크립트 추가 예정

```

스크립트에 대한 설명은 다음과 같습니다.  

- name
	- workflow 의 이름을 지정합니다.  
- on
	- 이 workflow 가 언제 실행될건지 트리거를 지정할 수 있습니다. 특정 브랜치가 push 되는 경우, Pull Request 가 생성될 경우, 또는 crontab 문법으로 스케줄링을 걸 수도 있습니다. 
	- `workflow_dispatch` 는 수동으로 해당 workflow 를 실행시키겠다는 의미입니다.
	- push 등의 이벤트에 의해 자동으로 배포가 되기 보다는 사람이 수동으로 빌드/배포를 실행하는 것이 안전하다고 생각합니다.  
- job, steps
	- workflow 는 하나 혹은 그 이상의 job 을 가질 수 있고 각 job 은 여러 step 에 따라 단계를 나눌 수 있습니다.  
- runs-on
	- 해당 workflow 를 어떤 OS 환경에서 실행할 것인지 지정할 수 있습니다.

각 step의 내용은 찬찬히 살펴보시면 이해하기 어렵지 않습니다. 
`actions/행위 이름` 을 사용하는 step 은 앞서 보았던 marketplace 에서 미리 정의되어 있는 행위를 가져와서 사용한 것입니다.  

checkout 은 깃헙이 제공하는 워크스페이스 (이 workflow 를 실행하는 공간) 에서 내 저장소가 위치한 곳으로 이동한다고 생각하시면 됩니다.  
이후에는 java 를 셋업하고 gradlew 에 실행권한을 준 뒤 프로젝트를 build 하는 과정입니다.  

> 각 스크립트에서 사용하는 파일의 위치, 경로 등은 프로젝트의 상황에 따라 알맞게 수정하셔야 합니다.
> 예를 들어 멀티모듈 프로젝트의 경우, 해당 workflow 의 위치나 각 모듈의 최상단 위치를 고려하여 스크립트를 작성하셔야 합니다.  

스크립트를 저장한 후, 다시 Actions 탭으로 이동하면 workflow 를 실행할 수 있는 페이지가 보입니다.  
방금 `workflow_dispatch` 로 workflow 를 수동 실행하겠다고 스크립트를 작성했기 때문에, 아래와 같이 **Run workflow - 브랜치 선택**하여 Job 을 실행합니다.  

[image:ED61BF1A-ABFA-475F-8462-6C1161FCFBF5-379-000022B9B2162C8A/868877C7-6D68-4E9C-85F2-57C99C1516BC.png]

그러면 다음과 같이 방금 만들었던 간단한 프로젝트에 대해 빌드가 실행된 것을 Step 별로 확인할 수 있습니다.  
만약 스크립트가 실패했다면 어떤 Step에서 문제가 있었는지도 상세 로그를 통해 확인할 수 있습니다.  

또한 bash 명령어를 얼마든지 추가할 수 있기 때문에, 필요에 따라 echo 등으로 각 단계 별 상태를 확인하면서 진행할 수도 있겠네요.  


[image:A6B4D014-B1C1-4F37-B962-DBDB8460E46D-366-000019DBF67C6E79/스크린샷 2020-08-12 오후 10.07.37.png]

프로젝트 빌드 작업까지 완료되었습니다!  

---

## S3 에 빌드한 jar 업로드

### S3 bucket 생성하기

[image:BEDB94FC-12C9-4142-97ED-70F954D03FCA-412-00000931CCAC8EBA/45E877D3-F38E-4FA7-A723-CD6890AF1020.png]

다음은 빌드한 jar 파일을 S3 버킷에 업로드해 보겠습니다.  

S3 (Simple Storage Service) 는 다들 잘 아시겠지만, AWS 에서 지원하는 파일 스토리지 서비스입니다.  
용량도 무제한이기에 여러가지 파일들을 필요에 따라 업로드/다운로드 하면서 다양한 방법으로 활용할 수 있는데요.  
지금처럼 CI/CD 배포 플로우에 S3를 이용한다고 하면 보통 build 한 jar 파일을 업로드하고, 배포하는 쪽에서 다운로드하여 배포하는 방식을 많이 사용합니다.  

먼저 AWS 에 로그인하여 S3 서비스로 이동합니다.  
그리고 아래와 같이 새 S3 버킷을 만들어 봅시다.  

> 버킷은 S3의 저장 공간 단위라고 생각하시면 됩니다.  
> 하나의 버킷 내에서도 디렉토리를 나누어 파일을 관리할 수 있습니다.  


[image:23552D2C-A8C9-40E8-8D1A-823F87553EA7-412-00000A1961A12DBA/A2C18541-1533-49B1-8D76-A74147A1FDEB.png]

버킷 이름을 입력하고 다음으로 넘어가면, 다른 옵션들은 특별히 건드릴 것이 없습니다.  

참고로 아래의 권한 설정에서는, `모든 퍼블릭 엑세스 차단` 이 체크되어 있는지 확인합니다.  
'어? 외부에서 접근하려면 퍼블릭 권한을 줘야 하는 것 아닌가?' 라고 생각하실 수도 있지만, 우리는 `IAM` 이라는 또 다른 AWS 의 권한 서비스를 이용하여 최소한의 접근 권한만 가지고 플로우를 구축할 것이기 때문에 넘어가시면 됩니다.  

[image:A2BB926B-84F2-4ACD-9FC8-2298EAFC9924-412-00000A2A909F18C0/B91D980E-1D98-4C37-AD52-7BA4D316ED12.png]

새로운 버킷이 만들어졌습니다!  

[image:31167FAE-671F-4736-ABBB-041264E5DDC3-412-00000AB495B82CA7/66AAF58B-98A6-4135-9093-E3184CD8C7E6.png]



### IAM 권한 부여하기

이제 Github Actions 가 해당 S3 버킷에 접근할 수 있도록 권한을 생성해 보겠습니다.  

IAM 은 AWS 서비스에 대한 엑세스 권한을 최소한으로 부여하고 관리할 수 있게 해주는 서비스입니다.  
외부 서비스에서 AWS 서비스에 접근하려고 할 때도 물론이고, AWS 서비스 간에 접근하고자 할 때도 권한 부여를 해줄 수 있습니다.  

우리는 Github Actions 에게 S3와 (나중 단계에서 사용할) CodeDeploy 에 대한 접근 권한을 가지고 있는 `사용자 역할`을 부여하려고 합니다.  
IAM 서비스로 이동해 다음과 같이 엑세스 관리 - 사용자 - 사용자 추가 를 선택하고 해당 사용자의 이름을 입력합니다.  
이 때 중요한 것은  `프로그래밍 방식 엑세스` 에 체크하는 것입니다.  
Github Actions 에서 AWS CLI 명령을 통해 엑세스할 것이기 때문에, 엑세스 키 ID, 비밀 엑세스 키를 발급받기 위해서 입니다.  
(엑세스 키 ID와 비밀 엑세스 키는 일종의 아이디/비밀번호 라고 생각하시면 됩니다.)


[image:C22642E3-82C2-4876-A171-F72F979FB126-412-00000B847E4270A8/70E369E5-C9B5-4EBE-952B-F11C1BA318B2.png]

다음은 부여할 권한 정책을 선택합니다.  
기존 정책 직접 연결 - 검색 에서 `AmazonS3FullAccess` 와 `AWSCodeDeployFullAccess` 를 선택합니다.  

[image:86DF58E0-4D3A-4A96-8B00-CA16D415F272-412-00000B7142928F15/38490D63-E30B-4B02-942A-D46451413E01.png]

태그는 다른 사용자 역할과 구분하기 위함이니 편하신대로 입력하시면 됩니다.  

[image:57A67958-51DD-4290-A421-B50156BA2D0D-412-00000BF958EFBBC5/1D2B9173-3DE2-47AD-85C9-4F056868291D.png]

사용자를 생성하면 마지막 페이지에서 다음과 같이 엑세스 키 ID와 비밀 엑세스 키가 나오는데요.  
이 값을 잘 복사해놓으시거나 왼편의 CSV 를 다운로드하여 보관하시기 바랍니다.  

[image:4496DD08-4739-49CE-9921-387E65EB8786-412-00000C39E3DBB1B6/2A7F7EC7-42ED-40F9-8B65-E3882BAD057F.png]


### S3 에 jar 파일 업로드하기

이제 만들어놓은 S3 버킷에 build 한 jar 파일을 업로드해 보겠습니다.  
업로드할 때는 zip 으로 압축하는 과정이 필요합니다.  

Github Actions 의 스크립트에 아래 내용을 추가하겠습니다.  

```yml
# logging-deploy.yml

on:
  workflow_dispatch:

env: # 새로 추가한 부분
  S3_BUCKET_NAME: logging-system-deploy
  PROJECT_NAME: playground-logging

jobs:
  build:
    runs-on: ubuntu-latest

    steps:

    # 중략

    ### 새로 추가한 부분 ###
    - name: Make zip file
      run: zip -r ./$GITHUB_SHA.zip .
      shell: bash

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ secrets.AWS_REGION }}

    - name: Upload to S3
      run: aws s3 cp --region ap-northeast-2 ./$GITHUB_SHA.zip s3://$S3_BUCKET_NAME/$PROJECT_NAME/$GITHUB_SHA.zip

```

- env
	- 현재 스크립트에서 사용할 환경변수를 정의하여 사용할 수 있습니다.
	- 여러 곳에서 공통으로 사용되는 문자열이나, 명확하게 의미를 부여해야 하는 키워드에 사용하시면 됩니다.
- `$GITHUB_SHA`
	- Github Actions 에서 제공하는 여러 기본 환경변수 중 하나입니다. 현재 workflow 를 실행시키는 커밋의 해쉬값입니다.  
	- [다른 기본 환경변수들](https://docs.github.com/en/actions/configuring-and-managing-workflows/using-environment-variables#default-environment-variables)도 필요에 따라 사용할 수 있습니다.  
- `${{ secrets.KEY_이름 }}`
	- 위의 환경변수와는 비슷하면서도 조금 다른 Context 입니다. Github Actions 에서 `${{ }}` 문법으로 여러 함수나 표현식, 값들을 등록하고 가져와서 사용할 수 있습니다. 자세한 내용은 [Contexts and Expressions - 레퍼런스](https://docs.github.com/en/actions/reference/context-and-expression-syntax-for-github-actions#contexts)를 참조해 보세요.
	- secrets 는 저장소에 등록한 비밀 키값을 가져오는 키워드인데요, 아래에서 설명하겠습니다.
- `aws s3 cp`
	- aws cli 명령어 중 하나입니다. copy 명령어로 현재 위치의 파일을 S3로 업로드하거나, 반대로 S3의 파일을 현재 위치로 다운로드할 수 있습니다. ([AWS CLI 레퍼런스 - 객체 관리](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-services-s3-commands.html#using-s3-commands-managing-objects))

jar 파일을 압축하고, AWS credential 을 설정하여 S3에 업로드하는 과정까지 추가가 되었는데요.  
아까 IAM 사용자 권한을 만들고 받았던 엑세스 키 ID와 비밀 엑세스 키를 해당 저장소에 등록해 보겠습니다.  


[image:B6ED50EF-04CD-420E-B16F-EFE6B26E49F0-412-00000D2B5EDA9BC9/A1A1F922-588A-4D90-AE48-7BB8236DFEA1.png]

Github 저장소의 Settings - Secrets - New secret 으로 스크립트에 작성했던 세 가지 값을 아래와 같이 등록합니다.  

```md
- AWS_ACCESS_KEY_ID : 엑세스 키 ID
- AWS_SECRET_ACCESS_KEY : 비밀 엑세스 키
- AWS_REGION : ap-northeast-2
```

저장한 비밀값들은 스크립트에서 `${{ secrets.KEY값 }}`으로 참조할 수 있습니다.  

다시 한 번 스크립트를 실행해보시고, S3 버킷을 열어보시면!  

[image:D5E97594-662A-40B5-A165-58710AB0C680-412-00000D56FBB752E8/62565CDD-4802-455E-955C-E0A66D019993.png]

해당 버킷에 프로젝트 이름으로 디렉토리가 생겼고, 안에는 프로젝트의 마지막 커밋 번호가 파일명인 zip 파일이 생긴 것을 확인하실 수 있습니다.  

---

## CodeDeploy

### 소개

[image:7039AA6A-16E4-4438-83B7-699CBEFB4E3B-368-000015551279EBE1/52E9F243-452B-4AE9-96FB-0B82F22FD405.png]

다음으로는 Github Actions 에서 CodeDeploy 에게 **S3에 있는 jar 파일을 가져가서 담당한 배포 그룹의 EC2에 배포해 줘!** 라는 명령을 내릴 수 있도록 구성해 보겠습니다.  

먼저 CodeDeploy에 대해 간단하게 소개하자면, 애플리케이션 배포를 자동화하는 AWS 의 배포 서비스입니다.  
EC2, AWS Lambda 와 같은 서비스에 배포를 할 수 있고, 현재위치 배포나 블루/그린 배포와 같은 무중단 배포를 지원합니다.  
한 번 구축해 놓으면 이후로는 배포가 매우 간편하고 AWS 콘솔을 통해 제어하면서 배포 과정을 확인할 수 있기 때문에 많은 분들이 이 서비스를 이용하여 배포 플로우를 구축합니다.  
조금 더 자세한 설명은 [CodeDeploy 레퍼런스](https://docs.aws.amazon.com/ko_kr/codedeploy/latest/userguide/welcome.html)를 참고해보시면 좋습니다.  

### 배포할 EC2 세팅하기


### CodeDeploy 배포 설정하기




---

## Nginx

```script
include /home/ec2-user/service-url.inc;

location / {
	proxy_set_header    X-Forwarded-For $remote_addr;
	proxy_set_header    Host $http_Host;
	proxy_pass          $service_url;
}
```















































