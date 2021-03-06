- 블로그 업로드
  - https://wbluke.tistory.com/39
  - https://wbluke.tistory.com/40
  - https://wbluke.tistory.com/41

---

# Github Actions + CodeDeploy + Nginx 로 무중단 배포하기
#TIL/devOps

---

## 개요

안녕하세요!  
이번 시리즈에서는 제목에서와 같이 Github Actions 와 CodeDeploy, 그리고 Nginx 를 사용하여 **하나의 서버에서 최소 규모의 무중단 배포**를 진행하는 방법에 대해 정리해보려고 합니다.  
관련 코드는 [Github 저장소](https://github.com/wbluke/playground) 에서 확인하실 수 있습니다.  


### 전체 흐름도

CI/CD 와 같이 인프라, 배포 환경을 구축하기 위해서는 내가 만들고자 하는 전체 그림을 숙지하는 것이 가장 중요하다고 생각합니다.  
그래야 진행~~삽질~~ 중에 막혔을 때 어느 부분이 문제일지 빠르게 유추해 볼 수 있기 때문입니다.  

![](./images/ACN_01.png)

이 그림이 지금부터 하나씩 만들어 볼 배포 플로우입니다.  

- Github Actions에서 프로젝트 빌드 후, jar 파일을 압축해서 S3에 업로드합니다.  
- 이어서 CodeDeploy에게 S3에 있는 jar 파일을 가지고 배포를 진행해달라고 전달합니다.
- CodeDeploy는 배포할 EC2 인스턴스 내부에 있는 CodeDeploy Agent에게 배포 명령을 내리고, Agent는 jar 파일을 S3에서 받아서 주어진 스크립트에 따라 배포를 진행합니다. 
- 새로운 Spring Boot WAS를 띄우고, Nginx 스위칭을 통해 무중단 배포를 할 수 있도록 Agent에게 배포 스크립트를 제공합니다.  

차근차근 진행해보도록 하겠습니다!  

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

```yaml
# application.yml

logging-module:
  version: 0.0.1
```

> 루트("/")로 접속했을 때의 화면이나 health-check 용으로 사용할 API 는 자유롭게 만드시면 됩니다.
> 이 프로젝트의 주된 목적은 아니니까요!

아래와 같이 정말 간단한 형태의 프로젝트가 완성되었습니다!

![](./images/ACN_02.png)


## Github Actions

### 소개

가장 먼저 구축해 볼 것은 [GitHub Actions](https://github.com/features/actions) 입니다.  

빌드와 배포를 유발하는 방법에는 정말 여러가지가 있을 수 있는데요, 작년 2019년에 갓 나온 Github Actions 는 손쉽게 개인 Github 코드 저장소에서 빌드/배포 환경을 구축할 수 있는 도구입니다.  
지금 당장 사용할 일이 없더라도 한번쯤은 사용해보고 배워두면 깃헙 저장소를 계속 이용하는 이상 정말 쓸모가 많을 것 같다는 생각이 드네요.  
심지어 public 저장소에서는 무료! 로 지원합니다.  

기본적으로 yml 파일로 스크립트를 짜기 때문에 해당 파일 형식에 익숙하시다면 금방 적응하실 수 있습니다.  

### workflow 생성하기

먼저 저장소의 Actions 에서 `set up a workflow yourself` 로 새 작업(이하 workflow)을 만들어 봅시다.  

![](./images/ACN_03.png)

새 workflow 를 생성하면 좌측에는 yml 스크립트를 입력할 수 있는 부분이 있고, 우측에는 미리 만들어져 있는 스크립트를 필요에 따라 추가할 수 있는 `Marketplace` 와 workflow 에 대한 설명을 볼 수 있는 `Documentation` 이 있습니다.  
Documentation 에는 비교적 쉽게 workflow 의 소개가 되어 있어서 한 번 참고해보시면 좋을 것 같습니다.  

그리고 눈치 채신 분들도 계시겠지만 workflow yml 파일은 상단에서 볼 수 있다시피 `프로젝트/.github/workflows/` 위치에 생성됩니다.  

![](./images/ACN_04.png)

좌측에 예시로 나와있는 설명도 간단하게 훑어보셨다면, 다음과 같이 스크립트를 작성해 보겠습니다.  

```yaml
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

![](./images/ACN_05.png)

그러면 다음과 같이 방금 만들었던 간단한 프로젝트에 대해 빌드가 실행된 것을 Step 별로 확인할 수 있습니다.  
만약 스크립트가 실패했다면 어떤 Step에서 문제가 있었는지도 상세 로그를 통해 확인할 수 있습니다.  

또한 bash 명령어를 얼마든지 추가할 수 있기 때문에, 필요에 따라 echo 등으로 각 단계 별 상태를 확인하면서 진행할 수도 있겠네요.  

![](./images/ACN_06.png)

프로젝트 빌드 작업까지 완료되었습니다!  

---

## S3 에 빌드한 jar 업로드

### S3 bucket 생성하기

![](./images/ACN_07.png)

다음은 빌드한 jar 파일을 S3 버킷에 업로드해 보겠습니다.  

S3 (Simple Storage Service) 는 다들 잘 아시겠지만, AWS 에서 지원하는 파일 스토리지 서비스입니다.  
용량도 무제한이기에 여러가지 파일들을 필요에 따라 업로드/다운로드 하면서 다양한 방법으로 활용할 수 있는데요.  
지금처럼 CI/CD 배포 플로우에 S3를 이용한다고 하면 보통 build 한 jar 파일을 업로드하고, 배포하는 쪽에서 다운로드하여 배포하는 방식을 많이 사용합니다.  

먼저 AWS 에 로그인하여 S3 서비스로 이동합니다.  
그리고 아래와 같이 새 S3 버킷을 만들어 봅시다.  

> 버킷은 S3의 저장 공간 단위라고 생각하시면 됩니다.  
> 하나의 버킷 내에서도 디렉토리를 나누어 파일을 관리할 수 있습니다.  


![](./images/ACN_08.png)

버킷 이름을 입력하고 다음으로 넘어가면, 다른 옵션들은 특별히 건드릴 것이 없습니다.  

참고로 아래의 권한 설정에서는, `모든 퍼블릭 엑세스 차단` 이 체크되어 있는지 확인합니다.  
'어? 외부에서 접근하려면 퍼블릭 권한을 줘야 하는 것 아닌가?' 라고 생각하실 수도 있지만, 우리는 `IAM` 이라는 또 다른 AWS 의 권한 서비스를 이용하여 최소한의 접근 권한만 가지고 플로우를 구축할 것이기 때문에 넘어가시면 됩니다.  

![](./images/ACN_09.png)

새로운 버킷이 만들어졌습니다!  

![](./images/ACN_10.png)



### IAM 권한 사용자 생성하기

이제 Github Actions 가 해당 S3 버킷에 접근할 수 있도록 권한을 생성해 보겠습니다.  

IAM 은 AWS 서비스에 대한 엑세스 권한을 최소한으로 부여하고 관리할 수 있게 해주는 서비스입니다.  
외부 서비스에서 AWS 서비스에 접근하려고 할 때도 물론이고, AWS 서비스 간에 접근하고자 할 때도 권한 부여를 해줄 수 있습니다.  

우리는 Github Actions 에게 S3와 (나중 단계에서 사용할) CodeDeploy 에 대한 접근 권한을 가지고 있는 `사용자 역할`을 부여하려고 합니다.  
IAM 서비스로 이동해 다음과 같이 엑세스 관리 - 사용자 - 사용자 추가 를 선택하고 해당 사용자의 이름을 입력합니다.  
이 때 중요한 것은  `프로그래밍 방식 엑세스` 에 체크하는 것입니다.  
Github Actions 에서 AWS CLI 명령을 통해 엑세스할 것이기 때문에, 엑세스 키 ID, 비밀 엑세스 키를 발급받기 위해서 입니다.  
(엑세스 키 ID와 비밀 엑세스 키는 일종의 아이디/비밀번호 라고 생각하시면 됩니다.)


![](./images/ACN_11.png)

다음은 부여할 권한 정책을 선택합니다.  
기존 정책 직접 연결 - 검색 에서 `AmazonS3FullAccess` 와 `AWSCodeDeployFullAccess` 를 선택합니다.  

![](./images/ACN_12.png)

태그는 다른 사용자 역할과 구분하기 위함이니 편하신대로 입력하시면 됩니다.  

![](./images/ACN_13.png)

사용자를 생성하면 마지막 페이지에서 다음과 같이 엑세스 키 ID와 비밀 엑세스 키가 나오는데요.  
이 값을 잘 복사해놓으시거나 왼편의 CSV 를 다운로드하여 보관하시기 바랍니다.  

![](./images/ACN_14.png)


### S3 에 jar 파일 업로드하기

이제 만들어놓은 S3 버킷에 build 한 jar 파일을 업로드해 보겠습니다.  
업로드할 때는 zip 으로 압축하는 과정이 필요합니다.  

Github Actions 의 스크립트에 아래 내용을 추가하겠습니다.  

```yaml
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


![](./images/ACN_15.png)

Github 저장소의 Settings - Secrets - New secret 으로 스크립트에 작성했던 세 가지 값을 아래와 같이 등록합니다.  

```md
- AWS_ACCESS_KEY_ID : 엑세스 키 ID
- AWS_SECRET_ACCESS_KEY : 비밀 엑세스 키
- AWS_REGION : ap-northeast-2
```

저장한 비밀값들은 스크립트에서 `${{ secrets.KEY값 }}`으로 참조할 수 있습니다.  

다시 한 번 스크립트를 실행해보시고, S3 버킷을 열어보시면!  

![](./images/ACN_16.png)

해당 버킷에 프로젝트 이름으로 디렉토리가 생겼고, 안에는 프로젝트의 마지막 커밋 번호가 파일명인 zip 파일이 생긴 것을 확인하실 수 있습니다.  

---

## CodeDeploy

### 소개

![](./images/ACN_17.png)

다음으로는 Github Actions 에서 CodeDeploy 에게 **S3에 있는 jar 파일을 가져가서 담당한 배포 그룹의 EC2에 배포해 줘!** 라는 명령을 내릴 수 있도록 구성해 보겠습니다.  

먼저 CodeDeploy에 대해 간단하게 소개하자면, 애플리케이션 배포를 자동화하는 AWS 의 배포 서비스입니다.  
EC2, AWS Lambda 와 같은 서비스에 배포를 할 수 있고, 현재위치 배포나 블루/그린 배포와 같은 무중단 배포를 지원합니다.  
한 번 구축해 놓으면 이후로는 배포가 매우 간편하고 AWS 콘솔을 통해 제어하면서 배포 과정을 확인할 수 있기 때문에 많은 분들이 이 서비스를 이용하여 배포 플로우를 구축합니다.  

> 조금 더 자세한 설명은 [CodeDeploy 레퍼런스](https://docs.aws.amazon.com/ko_kr/codedeploy/latest/userguide/welcome.html)를 참고해보시면 좋습니다.  

CodeDeploy 의 배포 과정은 다음과 같습니다.  

- 개발한 애플리케이션 최상단 경로에 AppSpec.yml 이라는 파일을 추가합니다.  
	* AppSpec.yml 은 배포에 필요한 모든 절차를 적어둔 명세서라고 생각하시면 됩니다.  
- CodeDeploy에 프로젝트의 특정 버전을 배포해 달라고 요청하면, CodeDeploy는 배포를 진행할 EC2 인스턴스에 설치돼 있는 CodeDeploy Agent들과 통신하며 Agent들에게 요청받은 버전을 배포해 달라고 요청합니다.
- 요청 받은 Agent들은 코드 저장소에서 프로젝트 전체를 서버에 내려받고, AppSpec.yml 파일을 읽어 해당 파일에 적힌 절차대로 배포를 진행합니다.
- Agent는 배포를 진행한 후 CodeDeploy에게 성공/실패 등의 결과를 알려줍니다.

CodeDeploy Agent 는 EC2 인스턴스에 설치되어 CodeDeploy의 명령을 기다리고 있는 프로그램입니다.  
실제로 배포를 수행하는 것은 Agent이기 때문에 반드시 EC2 인스턴스에 설치되어 있어야 합니다.  


### 배포할 EC2 세팅하기

먼저 우리가 애플리케이션을 배포할 EC2 를 하나 생성하겠습니다.  
EC2 인스턴스를 띄우는 방법은 이미 많은 자료가 있기 때문에 생략하겠습니다.  

> 저는 **Amazon Linux 2 AMI**를 사용하였습니다.
> Ubuntu 를 사용하시는 분들은 아래 소개되는 과정들에서 진행하는 커맨드가 조금 다를 수 있습니다. (패키지 매니저 등)

다만 다음과 같이 인바운드 규칙을 생성해주세요!  
ssh 접속은 현재 내 IP, 그리고 기본적으로 웹 서비스를 할 서버이기 때문에 HTTP, HTTPS 포트인 80, 443 포트에 대해서는 열어주시면 됩니다.    

![](./images/ACN_18.png)

ssh 로 새롭게 생성한 인스턴스에 접속해주세요!  

![](./images/ACN_19.png)

먼저 Java 가 있는지 확인하고 없다면 설치해줍니다.  

```sh
sudo yum install -y java-1.8.0-openjdk-devel.x86_64
java -version # 1.8 버전이 설치되었는지 확인
```

다음으로는 CodeDeploy Agent 를 설치하겠습니다.  

[CodeDeploy Agent 설치 레퍼런스](https://docs.aws.amazon.com/ko_kr/codedeploy/latest/userguide/codedeploy-agent-operations-install-linux.html)를 참고하셔서 다음 커맨드를 차례로 수행합니다.  

```sh
# 패키지 매니저 업데이트, ruby 설치
sudo yum update
sudo yum install ruby
sudo yum install wget

# 서울 리전에 있는 CodeDeploy 리소스 키트 파일 다운로드
cd /home/ec2-user
wget https://aws-codedeploy-ap-northeast-2.s3.ap-northeast-2.amazonaws.com/latest/install

# 설치 파일에 실행 권한 부여
chmod +x ./install

# 설치 진행 및 Agent 상태 확인
sudo ./install auto
sudo service codedeploy-agent status
```


![](./images/ACN_20.png)

CodeDeploy Agent 가 실행되었음을 확인할 수 있습니다!  

만약 status 확인 시 Agent 가 실행중이지 않다면 다음 커맨드를 수행합니다.  

```sh
sudo service codedeploy-agent start
```


### EC2 에 IAM 역할 부여하기

이전 글에서 IAM 권한 사용자를 생성하여 Github Actions 가 해당 사용자의 권한을 사용할 수 있도록 설정한 것을 기억하실텐데요.  
이번에는 EC2 에 S3, CodeDeploy 권한 정책을 부여해 보겠습니다.  
IAM 사용자는 아이디/비밀번호 기반으로 외부 프로그램이 사용자처럼 접근하는 것이라면, IAM 역할은 다른 계정의 IAM 사용자, 혹은 다른 AWS 서비스가 수행하는 역할 권한을 부여하는 것이라고 생각하시면 됩니다.  

다음과 같이 IAM 의 역할로 이동하여 역할을 만들어 보겠습니다.  

![](./images/ACN_21.png)

사용 사례에서 EC2 를 선택합니다.  

![](./images/ACN_22.png)

S3 와 CodeDeploy 에 대한 권한 정책을 검색하여 추가합니다.  

![](./images/ACN_23.png)

적당한 태그와 역할 이름을 설정하고 역할 만들기를 선택합니다.  

![](./images/ACN_24.png)

역할을 생성했다면 EC2 대시보드로 돌아와 해당 EC2 우클릭 - 인스턴스 설정 - IAM 역할 연결/바꾸기 를 선택합니다.  

![](./images/ACN_25.png)

방금 생성했던 IAM 역할을 부여합니다.  

![](./images/ACN_26.png)


### CodeDeploy IAM 역할 생성하기

이제 CodeDeploy를 설정해 보겠습니다.  

CodeDeploy 애플리케이션을 생성하기 전에, 앞에서와 마찬가지로 CodeDeploy를 위한 IAM Role이 필요합니다.  
다시 IAM - 역할 - 역할 만들기 로 이동하여, 아래쪽의 CodeDeploy 선택, 사용 사례도 CodeDeploy를 선택하고 다음으로 넘어갑니다.  

![](./images/ACN_27.png)

나머지는 방금전까지 했던 과정과 동일합니다.  
역할의 이름을 정하고 역할을 생성합니다.  

![](./images/ACN_28.png)


### CodeDeploy 애플리케이션 생성하기

다음으로 CodeDeploy 애플리케이션을 생성하겠습니다.  
CodeDeploy - 애플리케이션 - 애플리케이션 생성 에서 이름과 EC2를 선택하고 생성을 진행합니다.  

![](./images/ACN_29.png)

다음으로는 만들어진 애플리케이션에서 **배포 그룹 생성**을 클릭합니다.  

![](./images/ACN_30.png)

배포 그룹 생성 화면에서 이름과, 방금 만들었던 IAM 역할을 연결합니다.  
우리는 최소 단위의 무중단 배포를 CodeDeploy가 아닌 Nginx를 통해 구현할 것이기 때문에, 지금은 일단 현재 위치 배포를 선택하고 넘어갑니다.  

> 현재 위치 배포와 블루/그린 배포 모두 무중단 배포의 형태들입니다.  
> 다만 지금 우리는 서버를 여러 대 띄우는 방식이 아니라, 하나의 EC2 내부에서 두 개의 포트에 WAS를 띄우고 스위칭하는 방식을 쓸 것이기 때문에 CodeDeploy가 무중단 배포의 역할을 가지고 있지는 않습니다.  


![](./images/ACN_31.png)

환경 구성은 EC2를 선택하고 태그에는 EC2 인스턴스에 설정했던 태그를 걸어줍니다.  
해당 태그를 기준으로 CodeDeploy가 배포를 진행합니다.  

![](./images/ACN_32.png)


배포 설정은 여러 대의 서버를 어떤 단계에 따라 순차적으로 배포할 것인지를 선택하는 설정인데요.  
어차피 EC2가 하나기 때문에 한번에 배포를 끝내는  `CodeDeployDefault.AllAtOnce` 를 선택합니다.  

로드밸런서는 없기 때문에 활성화를 해제하고 배포 그룹을 생성합니다.  

![](./images/ACN_33.png)


### 스크립트 추가하기

이제 CodeDeploy의 설정은 끝이 났습니다!  
마지막으로 프로젝트에서 스크립트를 몇 가지 추가해 보겠습니다.  

먼저 프로젝트의 최상단에 `appspec.yml`을 생성합니다.  
appspec.yml은 CodeDeploy Agent가 참조하면서 배포를 진행하는 명세서입니다.  

![](./images/ACN_34.png)

기본적인 appspec.yml을 다음과 같이 작성합니다.  
다음 단계인 Nginx 적용 단계에서 스크립트를 추가적으로 작성하겠습니다.  

```yaml
# appspec.yml

version: 0.0
os: linux
files:
  - source: /
    destination: /home/ec2-user/playground-logging/ # 프로젝트 이름
    overwrite: yes

permissions:
  - object: /
    pattern: "**"
    owner: ec2-user
    group: ec2-user
```

- files.destination
	- S3에서 받아온 프로젝트의 위치를 지정해주시면 됩니다.  

그리고 Github Actions의 스크립트에도 마지막 단계를 업데이트합니다.  
Github Actions 는 이번 CodeDeploy Step 작업을 끝으로 더이상 추가할 것이 없으니 전체 스크립트를 다시 보여드리겠습니다.  

```yaml
# logging-deploy.yml

name: logging-system

on:
  workflow_dispatch:

env:
  S3_BUCKET_NAME: logging-system-deploy
  PROJECT_NAME: playground-logging

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
        
      ### 새로 추가한 부분 ###
      - name: Code Deploy
        run: aws deploy create-deployment --application-name logging-system-deploy --deployment-config-name CodeDeployDefault.AllAtOnce --deployment-group-name develop --s3-location bucket=$S3_BUCKET_NAME,bundleType=zip,key=$PROJECT_NAME/$GITHUB_SHA.zip
```

- application-name
	- CodeDeploy 애플리케이션의 이름을 지정합니다.
- deployment-config-name
	- 배포 그룹 설정에서 선택했던 배포 방식을 지정합니다.
- deployment-group-name
	- 배포 그룹의 이름입니다.
- s3-location
	- jar를 S3에서 가지고 오기 위해 차례로 bucket 이름, 파일 타입, 파일 경로를 입력합니다.  

여기까지가 이번 단계의 끝입니다!  

Github Actions에서 배포를 실행하기 전에 한 가지 진행해야 하는 부분이 있는데요.  
지금까지의 과정 순서를 똑같이 따라하셨다면, CodeDeploy Agent를 시작한 후에 IAM Role을 EC2에 부여했기 때문에 아마 Agent가 IAM Role을 가지고 있지 않아 배포가 실패할 것입니다.  

EC2에 접속해서 다음 커맨드로 Agent를 재시작해주세요!  

```sh
sudo service codedeploy-agent restart
```

그리고 나서 Github Actions의 workflow를 시작하고 잠시 기다리면!  

![](./images/ACN_35.png)

CodeDeploy의 배포가 성공한 것을 볼 수 있습니다!  

EC2에 접속해서 한번 확인해 보겠습니다.  

![](./images/ACN_36.png)

ec2-user home 디렉토리에 S3에서 받아온 프로젝트가 있는 것을 확인할 수 있습니다!  
물론 아직 진짜 배포를 하도록 스크립트를 짜지는 않았지만, 적어도 각 서비스 간 통신은 잘 이루어졌다고 생각할 수 있습니다.  

만약 배포가 실패했다면 아래쪽에 배포 실행 목록에서 `View events`를 확인하면 에러 로그를 볼 수 있습니다.  
[View events를 확인했는데 에러 로그도 보이지 않는다면, 이 글](https://wbluke.tistory.com/37)을 참고해 주세요!  

여기까지 완료했다면 이제 마지막 단계인 Nginx 무중단 배포를 진행하러 가보겠습니다.  

---

## Nginx

### 소개

![](./images/ACN_37.png)

Nginx는 널리 쓰이는 웹 서버 중 하나입니다.  
동적 처리를 주로 담당하는 WAS(Web Application Server)와는 다르게 웹 서버(Web Server)는 정적 자원에 대한 응답을 내려주는 역할을 가지고 있는데요.  
Nginx는 정적 자원의 처리 외에도 proxy 서버의 역할이나, reverse proxy 서버의 역할 등 여러방면에서 높은 활용도를 보여줍니다.  

여기서는 Nginx가 CodeDeploy Agent에 의해 두 WAS간의 스위칭 역할을 담당하도록 구성해 보겠습니다.  

### Nginx 설치와 설정

먼저 Nginx를 설치하겠습니다.  

EC2에 ssh로 접속하여 다음 커맨드를 수행합니다.  

```sh
sudo yum install nginx
```

그럼 설치가 되는 듯 하였으나, 다음과 같이 Amazon Linux 에서의 설치법을 따로 안내해줍니다.  

![](./images/ACN_38.png)

안내대로 다시 커맨드를 입력합니다.  

```sh
sudo amazon-linux-extras install nginx1
sudo nginx -v # 설치 버전 확인
```

설치 후 `/etc/nginx/` 로 이동해 보시면 다양한 nginx 설치파일들을 보실 수 있는데요.  
우리가 관심있게 보아야 할 것은 설정파일인  `nginx.conf` 파일입니다.  

파일 수정이 필요하니 sudo 권한으로 nginx.conf 파일을 엽니다.  

```sh
sudo vim /etc/nginx/nginx.conf
```

다음과 같이 스크립트를 추가하겠습니다.  

![](./images/ACN_39.png)

```sh
include /home/ec2-user/service_url.inc;

location / {
	proxy_set_header    X-Forwarded-For $remote_addr;
	proxy_set_header    Host $http_Host;
	proxy_pass          $service_url;
}
```

- include
	- 다른 곳에 존재하는 설정 파일 등을 불러올 수 있습니다.  
- proxy_pass
	- 우리가 지정할 **$service_url**로 요청을 보낼 수 있도록 하는 프록시 설정입니다.  

include 로 불러올 파일 경로에 다음과 같이 파일을 생성하고 $service_url 변수를 설정하겠습니다.  

```sh
vim /home/ec2-user/service_url.inc
```

```sh
# service_url.inc

set $service_url http://127.0.0.1:8081;
```

이러면 nginx 설정은 끝입니다!  

다음 명령어로 nginx를 시작하고 nginx의 status를 확인할 수 있습니다.  

```sh
sudo service nginx start
sudo service nginx status
```


### 배포 스크립트 추가

마지막으로 프로젝트에 CodeDeploy Agent가 참고하여 배포를 진행하기 위한 스크립트들을 추가해보도록 하겠습니다.  

appspec.yml에 다음과 같이 스크립트를 추가합니다.  

```yaml
# appspec.yml

version: 0.0
os: linux
files:
  - source: /
    destination: /home/ec2-user/playground-logging/
    overwrite: yes

permissions:
  - object: /
    pattern: "**"
    owner: ec2-user
    group: ec2-user

### 새로 추가한 부분 ###
hooks:
  ApplicationStart:
    - location: scripts/run_new_was.sh
      timeout: 180
      runas: ec2-user
    - location: scripts/health_check.sh
      timeout: 180
      runas: ec2-user
    - location: scripts/switch.sh
      timeout: 180
      runas: ec2-user
```

- hooks
	- CodeDeploy의 배포에는 각 단계 별 수명 주기가 존재합니다. 수명 주기에 따라 원하는 스크립트를 수행할 수 있습니다.  
	- [AppSpec 'hooks' 섹션 레퍼런스](https://docs.aws.amazon.com/ko_kr/codedeploy/latest/userguide/reference-appspec-file-structure-hooks.html#reference-appspec-file-structure-hooks-list)

ApplicationStart라는 수명 주기에 세 가지 스크립트를 차례로 실행시키겠습니다.  

프로젝트 최상단에 scripts 라는 디렉토리를 만들고 다음과 같이 세 개의 파일을 만들겠습니다!  

![](./images/ACN_40.png)


```sh
# run_new_was.sh

#!/bin/bash

CURRENT_PORT=$(cat /home/ec2-user/service_url.inc | grep -Po '[0-9]+' | tail -1)
TARGET_PORT=0

echo "> Current port of running WAS is ${CURRENT_PORT}."

if [ ${CURRENT_PORT} -eq 8081 ]; then
  TARGET_PORT=8082
elif [ ${CURRENT_PORT} -eq 8082 ]; then
  TARGET_PORT=8081
else
  echo "> No WAS is connected to nginx"
fi

TARGET_PID=$(lsof -Fp -i TCP:${TARGET_PORT} | grep -Po 'p[0-9]+' | grep -Po '[0-9]+')

if [ ! -z ${TARGET_PID} ]; then
  echo "> Kill WAS running at ${TARGET_PORT}."
  sudo kill ${TARGET_PID}
fi

nohup java -jar -Dserver.port=${TARGET_PORT} /home/ec2-user/playground-logging/build/libs/* > /home/ec2-user/nohup.out 2>&1 &
echo "> Now new WAS runs at ${TARGET_PORT}."
exit 0
```

- 새로운 WAS를 띄우는 스크립트입니다.
	- service_url.inc 에서 현재 서비스를 하고 있는 WAS의 포트 번호를 읽어옵니다.
	- 현재 포트 번호가 8081이면 새로 WAS를 띄울 타겟 포트는 8082, 혹은 그 반대 상황이라면 8081을 지정합니다.  
	- 만약 타겟포트에도 WAS가 떠 있다면 kill하고 새롭게 WAS를 띄웁니다.
- nohup
	- 터미널 엑세스가 끊겨도 실행한 프로세스가 계속 동작하게 합니다.
	- 마지막의 `&`는 프로세스가 백그라운드로 실행되도록 해줍니다.  


```sh
# health_check.sh

#!/bin/bash

# Crawl current connected port of WAS
CURRENT_PORT=$(cat /home/ec2-user/service_url.inc | grep -Po '[0-9]+' | tail -1)
TARGET_PORT=0

# Toggle port Number
if [ ${CURRENT_PORT} -eq 8081 ]; then
	TARGET_PORT=8082
elif [ ${CURRENT_PORT} -eq 8082 ]; then
	TARGET_PORT=8081
else
	echo "> No WAS is connected to nginx"
	exit 1
fi


echo "> Start health check of WAS at 'http://127.0.0.1:${TARGET_PORT}' ..."

for RETRY_COUNT in 1 2 3 4 5 6 7 8 9 10
do
	echo "> #${RETRY_COUNT} trying..."
	RESPONSE_CODE=$(curl -s -o /dev/null -w "%{http_code}"  http://127.0.0.1:${TARGET_PORT}/health)

	if [ ${RESPONSE_CODE} -eq 200 ]; then
		echo "> New WAS successfully running"
		exit 0
	elif [ ${RETRY_COUNT} -eq 10 ]; then
		echo "> Health check failed."
		exit 1
	fi
	sleep 10
done
```

- 새로 띄운 WAS가 완전히 실행되기까지 health check 하는 스크립트입니다.  


```sh
# switch.sh

#!/bin/bash

# Crawl current connected port of WAS
CURRENT_PORT=$(cat /home/ec2-user/service_url.inc  | grep -Po '[0-9]+' | tail -1)
TARGET_PORT=0

echo "> Nginx currently proxies to ${CURRENT_PORT}."

# Toggle port number
if [ ${CURRENT_PORT} -eq 8081 ]; then
	TARGET_PORT=8082
elif [ ${CURRENT_PORT} -eq 8082 ]; then
	TARGET_PORT=8081
else
	echo "> No WAS is connected to nginx"
	exit 1
fi

# Change proxying port into target port
echo "set \$service_url http://127.0.0.1:${TARGET_PORT};" | tee /home/ec2-user/service_url.inc

echo "> Now Nginx proxies to ${TARGET_PORT}."

# Reload nginx
sudo service nginx reload

echo "> Nginx reloaded."
```

- nginx 리로드를 통해 서비스하는 포트를 스위칭하는 스크립트입니다. 
	- `sudo service nginx reload` 는 nginx 서버의 재시작 없이 바로 새로운 설정값으로 서비스를 이어나갈 수 있도록 합니다.
	- `sudo service nginx restart` 는 말그대로 서버의 shutdown 이후 재시작하는 명령이므로 의도하지 않았다면 주의해야 합니다.
- tee
	- 출력 내용을 파일로 만들어주는 커맨드입니다.
	- 새로 띄운 WAS의 포트를 nginx가 읽을 수 있도록 service_url.inc에 내용을 덮어씁니다. 


이제 모든 준비가 끝났습니다!  
무중단 배포를 진행하기 전에, 현재 서버에 아무런 WAS가 떠 있지 않기 때문에 8081 포트에 WAS를 새로 한번 띄워보겠습니다.  

EC2에 접속하여 프로젝트 내에 있는 jar 파일을 실행하겠습니다.  

```sh
nohup java -jar -Dserver.port=8081 /home/ec2-user/playground-logging/build/libs/* &
```

nginx가 바라보도록 설정한 service_url.inc 파일이 8081포트를 가리키고 있기 때문에, 8081로 서버를 띄운 후 EC2 인스턴스의 퍼블릭 DNS로 접속해 보시면 다음과 같이 우리가 만들었던 Controller의 리턴값이 잘 나오는 것을 볼 수 있습니다.  

![](./images/ACN_41.png)

이제 드디어 무중단 배포를 진행해볼 차례입니다.  

application.yml의 버전을 0.0.2로 올리고, 커밋 - 푸시 후 Github Actions에서 배포를 진행해봅시다.  

![](./images/ACN_42.png)

Github Actions 의 작업이 끝나고나서, CodeDeploy가 배포를 시작할 때 브라우저에서 지속적으로 새로고침을 눌러보시면!  

![](./images/ACN_43.png)

서버의 중단 없이 한순간에 버전이 바뀌는 것을 볼 수 있습니다! (드디어!)  

`ps -ef | grep java` 커맨드를 통해 배포 전과 배포 후를 비교해보면 8082 포트로 새로운 서버가 실행된 것을 볼 수 있습니다.  

![](./images/ACN_44.png)

그리고 `tail service_url.inc`로 내용을 확인해보시면 우리가 작성한 스크립트가 Nginx가 바라보는 포트를 8082로 변경한 것도 보실 수 있습니다!  

![](./images/ACN_45.png)

이제 그 다음 배포를 한번 더 진행한다고 하면, 새로운 서버는 기존 8081 포트의 애플리케이션을 종료하고 새로 뜰 것이고, Nginx는 8081 포트를 서비스할 것입니다.  

현재 이 배포 구성은 V2 버전의 WAS가 서비스를 하고 있으면서 이전 버전인 V1의 WAS가 서비스를 하지 않는데도 백그라운드에 그대로 떠 있는 상황인데요.  
배포 직후 모니터링 과정에서 롤백해야 하는 상황이 온다면 롤백용 스크립트를 추가해서 V1 버전의 WAS를 다시 서비스하도록 Nginx를 reload하는 방식을 적용할 수도 있습니다.  
또는 서비스 하는 내내 두 개의 WAS를 불필요하게 띄워놓을 필요가 없겠다는 생각이 드신다면, 배포 직후 V1 버전의 WAS가 롤백 대비용으로 떠 있다가, 모니터링을 진행하고 일정 시간이 지나면 해당 WAS를 종료하는 스크립트도 추가할 수 있을 것입니다.  

이번 Github Actions + CodeDeploy + Nginx 를 이용해 최소 규모로 무중단 배포하기 시리즈는 여기까지입니다!  
긴 글 읽어주셔서 감사합니다 :)  


