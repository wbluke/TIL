## VPC (Virtual Private Cloud)

[Amazon VPC란 무엇인가?](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/what-is-amazon-vpc.html)

### 보안 그룹 (Security Group)

EC2 인스턴스에 대한 방화벽 역할을 한다.  
인스턴스 수준에서 인바운드 트래픽과 아웃바운드 트래픽을 모두 제어한다.  

- 인스턴스 레벨에서 운영
- 허용 규칙만 지원
- 상태 저장: 규칙에 관계없이 반환 트래픽이 자동으로 허용됨
- 트래픽 허용 여부를 결정하기 전에 모든 규칙을 평가함
- 인스턴스 시작 시 누군가 보안 그룹을 지정하거나, 나중에 보안 그룹을 인스턴스와 연결하는 경우에만 인스턴스에 적용됨

### Network ACL

서브넷 간의 통신 권한을 관리한다.  

- 서브넷 레벨에서 운영
- 허용 및 거부 규칙 지원
- 상태 비저장: 반환 트래픽이 규칙에 의해 명시적으로 허용되어야 함
- 트래픽 허용 여부를 결정할 때 번호가 가장 낮은 규칙부터 순서대로 규칙을 처리
- 연결된 서브넷의 모든 인스턴스에 자동 적용됨(보안 그룹 규칙이 지나치게 허용적일 경우 추가 보안 계층 제공)

---

## Amazon ELB (Elastic Load Balancer)

[웹 서버 로드 밸런싱 | 서버 로드 밸런싱 | Amazon Web Services](https://aws.amazon.com/ko/elasticloadbalancing/)

트래픽이 몰릴 경우 Auto Scaling과 같은 작업을 하여 트래픽을 분산시키는 서비스.  
On Premise의 L4 Switch와 동일한 역할을 한다.  

AWS에서는 CLB(Classic Load Balancer), ALB(Application Load Balancer), NLB(Network Load Balancer) 총 3개의 로드벨런서가 있다. (출시순)  

SNI(Server Name Indication, 서버 이름 표시)를 사용해 다중 TLS/SSL 인증서 지원을 할 수 있다.  
이를 통해 단일 로드 밸런서 뒤에서 각각 자체 TLS 인증서를 갖는 다수의 TLS 보안 애플리케이션을 호스팅할 수 있다.  
SNI는 로드 밸런서에서 동일한 보안 리스너로 다수의 인증서를 바인딩하기만 하면 사용할 수 있는데, 그러면 ELB가 각 클라이언트마다 최적의TLS 인증서를 자동으로 선택한다.  

> SNI(Server Name Indication, 서버 이름 표시) : TLS 핸드쉐이킹 과정에서 클라이언트가 어느 호스트에 접속하려는지 서버에 알리는 역할

SAA 내용은 아니지만 참고하면 좋은 블로그. 내용이 좋다.  

[AWS Network Intro - Elastic Load Balancer](https://aws-diary.tistory.com/11)

---

## Amazon S3

[Amazon Simple Storage Service(S3) - 클라우드 스토리지 - AWS](https://aws.amazon.com/ko/s3/faqs/)

Amazon 스토리지 서비스.  

Amazon S3는 객체 생성/복제/제거/복원/손실 등의 알림을 Amazon SNS, Amazon SQS, AWS Lambda로 보낼 수 있다.  

### S3 스탠다드-Infrequent Access(S3 스탠다드-IA)

액세스 빈도가 낮지만 필요할 때 빠르게 액세스해야 하는 데이터를 위한 스토리지이다.  

### Amazon S3 One Zone-Infrequent Access(S3 One Zone-IA)

하나의 가용 영역 안에서만 중복 저장하여 다중 가용 영역에 데이터를 저장하는 스탠다드 IA 스토리지보다 20% 저렴하게 이용할 수 있다.  
99%의 가용성을 제공한다. (가용영역 내에서는 트웰브 나인 가용성)  
스탠다드와는 다르게 가용성 영역 파괴 시 데이터 손실된다.  

이런 이유로 자주 액세스 하지 않는 데이터를 적재할 때 사용한다.  

### Amazon S3 Glacier

Amazon S3 Glacier는 데이터 보관 및 장기 백업을 위한 안전하고 안정적이며 비용이 매우 저렴한 Amazon S3 스토리지 클래스.  

### Amazon S3 Glacier Deep Archive

Amazon S3 Glacier는 장기 보관용이지만 신속 검색을 사용하여 몇 분 안에 데이터에 접근할 수 있다.  
Amazon S3 Glacier Deep Archive는 장기 보관이면서 데이터에 거의 액세스하지 않는 경우에 훨씬 더 저렴하게 이용할 수 있다.  

### Amazon S3 Transfer Acceleration

거리가 먼 클라이언트와 Amazon S3 버킷 간에 파일을 빠르고, 쉽고, 안전하게 전송할 수 있게 해준다.  
전 세계적으로 분산된 Amazon CloudFront의 AWS 엣지 로케이션을 활용한다.  

### Amazon S3가 관리하는 암호화 키(SSE-S3)를 사용하는 서버 측 암호화로 데이터 보호

[Amazon S3가 관리하는 암호화 키(SSE-S3)를 사용하는 서버 측 암호화로 데이터 보호](https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/dev/UsingServerSideEncryption.html)

[REST API를 사용하여 고객 제공 암호화 키로 서버 측 암호화 지정](https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/dev/ServerSideEncryptionCustomerKeysSSEUsingRESTAPI.html)

S3 데이터 저장 시 버킷 정책에 `서버측 암호화`를 요청할 수 있다.  
Amazon S3 서버측 암호화는 256비트 고급 암호화 표준(AES-256)을 사용하여 데이터를 암호화한다.  
이 때 `x-amz-server-side-encryption` 헤더가 필요하다.  

반대로 고객 제공 암호화 키(SSE-C)를 사용할 수도 있다.  
다음과 같은 헤더가 쓰인다.  

- `x-amz-server-side-encryption-customer-algorithm`
    - 암호화 알고리즘 지정
- `x-amz-server-side-encryption-customer-key`
    - 이 헤더를 사용하여 256비트의 base64 인코딩 암호화 키를 Amazon S3에 제공하여 데이터를 암호화하거나 암호 해독
- `x-amz-server-side-encryption-customer-key-MD5`
    - 이 헤더를 사용하여 RFC 1321에 따라 암호화 키의 128비트 base64 인코딩 MD5 다이제스트를 제공

---

## AWS Storage Gateway

[AWS Storage Gateway(이)란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/storagegateway/latest/userguide/WhatIsStorageGateway.html)

클라우드 스토리지 서비스. 보통 백업 데이터를 받아서 뒤쪽의 S3로 넘기는 작업을 한다.  
즉, 온프레미스 환경에 있던 데이터를 일정 시간이 지나면 AWS로 옮긴다는 뜻이고, AWS DataSync와는 다르게 hybrid cloud storage를 구현할 때 사용한다.  

- 백업을 클라우드로 이동
- 온프레미스 파일 공유
- AWS의 데이터에 낮은 지연 시간으로 액세스 가능

파일 게이트웨이, 테이프 게이트웨이, 볼륨 게이트웨이의 3가지 게이트웨이를 제공한다.  

Storage Gateway는 대용량 데이터를 적재하는 데에 적합하지 않다.   
보통 변경된 데이터나 압축된 데이터에만 적절하다.  
대용량 데이터를 적재하고 싶다면 대신 AWS DataSync를 사용해야 한다.  

---

## AWS DataSync

[AWS DataSync이란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/datasync/latest/userguide/what-is-datasync.html)

간단하게 엄청나게 많은 양의 데이터를 온프레미스 환경에서 Amazon S3, Amazon EFS(Elastic File System), Windows 파일 서버를 위한 Amazon FSx 등으로 적재할 수 있다.  
보통 대량의 히스토리성 콜드 데이터들을 Amazon S3 Glacier나 Amazon S3 Glacier Deep Archive에 옮길 때 사용한다.  

---

## Amazon EFS (Elastic File System)

[Amazon Elastic File System란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/efs/latest/ug/whatisefs.html)

완전 관리형 파일 시스템.  
파일 추가/삭제에 따라 자동으로 용량이 확장/축소된다.  

---

## Amazon CloudFront

[Amazon CloudFront란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)

HTML, CSS, JS 및 이미지 파일과 같은 정적 및 동적 웹 콘텐츠를 사용자에게 더 빨리 배포하도록 지원하는 CDN(Content delivery network) 웹 서비스이다.  
S3에 오리진 데이터를 넣어놓고 CloudFront를 연결한 후 글로벌한 CloudFront 엣지 로케이션에 배포할 수 있다.  

---

## Amazon CloudWatch

[Amazon CloudWatch이란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)

실시간으로 실행 중인 애플리케이션을 여러가지 지표로 모니터링할 수 있는 도구.  

---

## Amazon SQS (Simple Queue Service)

[Amazon Simple Queue Service(이)란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html)

완전관리형 메시지 대기열 서비스.  

Amazon SQS는 최대 메시지 보존 기간을 넘겨 대기열에 존재해온 메시지를 삭제한다.  
기본 메시지 보존 기간(retention period)은 4일이다.  
하지만 설정을 통해 메시지 보존 기간을 60초에서 최대 14일까지로 변경할 수 있다.  

---

## Amazon SNS (Simple Notification Service)

[Amazon SNS란 무엇인가요?](https://docs.aws.amazon.com/ko_kr/sns/latest/dg/welcome.html)

Pub/Sub 구조에서 사용되는 메시지 전송 관리형 서비스.  

---

## Amazon Aurora

### instance cluster + endpoint

Amazon Aurora 는 전형적으로 단일 인스턴스가 아닌 인스턴스 군을 포함한다.  
각각의 커넥션은 특정한 DB 인스턴스에 의해 관리된다.  
Aurora cluster에 접근하면, 호스트 이름과 포트번호로 endpoint라 불리는 특정 핸들러를 지정받을 수 있다.  

Aurora에서는 각각의 인스턴스 군이 서로 다른 역할을 가져갈 수 있다.  
endpoint를 사용하면 각각의 커넥션을 역할에 맞는 인스턴스 군으로 매핑할 수 있다.  

---

# 기타 서비스

## Redshift

[데이터 웨어하우스 | Redshift | Amazon Web Services](https://aws.amazon.com/ko/redshift/?whats-new-cards.sort-by=item.additionalFields.postDateTime&whats-new-cards.sort-order=desc)

클라우드 데이터 웨어하우스. OLAP 환경에서 사용한다.  

## AWS LightSail

[Amazon Lightsail](https://aws.amazon.com/ko/lightsail/)

EC2보다 가볍고, 단순화 된 서비스.  
버스트 기능이 있는 t2 계열의 EC2 instance라 볼 수 있다.  
LightSail의 대상은 AWS의 EC2, EBS, VPC, 그리고 Route53 같은 무수한 옵션들을 고려하고 싶지 않은 “간단한 VPS를 원하는” 고객들이다.  

## Amazon Kinesis

[실시간 데이터 분석 처리 시스템 | Amazon Web Services](https://aws.amazon.com/ko/kinesis/)

실시간 비디오, 실시간 스트리밍 데이터를 수집, 처리, 분석할 수 있는 서비스.  

## Amazon SES (Simple Email Service)

[Amazon Simple Email Service | Cloud Email Service | Amazon Web Services](https://aws.amazon.com/ko/ses/)

애플리케이션에서 대규모 이메일을 글로벌하고 안전하게 보낼 수 있는 서비스.  

## Amazon SWF (Simple Workflow Service)

[AWS | Amazon Simple Workflow Service - 클라우드 워크플로 관리](https://aws.amazon.com/ko/swf/)

병렬 또는 순차 단계가 있는 백그라운드 작업을 구축하고 실행하고 확장할 수 있는 서비스.
