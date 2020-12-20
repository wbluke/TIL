## EC2 (Elastic Compute Cloud)

[Amazon EC2이란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/WindowsGuide/concepts.html)

EC2는 물리 서버의 기능을 가상화했지만 실제 서버와 유사하게 작동하며, 스토리지, 메모리, 네트워크 인터페이스, OS가 새로 설치된 기본 드라이브가 제공된다.  

### **EC2 AMI**

AMI(Amazon Machine Image)에는 다음과 같이 4가지 종류가 있다.  

- Amazon 빠른 시작 AMI
    - 다양한 리눅스 배포판, Windows Server OS, 딥러닝, 데이터베이스 등의 공통 작업을 위한 특수 이미지가 포함된다.
    - 가장 많이 사용하고 Amazon에서 항상 최신버전으로 공식 지원한다.
- AWS Marketplace AMI
    - 프로덕션 환경에서 사용 가능한 공식 이미지
- 커뮤니티 AMI
    - 특정한 요구에 맞도록 독립 공급업체가 제작하고 관리한다.
- 프라이빗 AMI
    - 사용자가 자체 배포한 인스턴스에서 이미지를 생성해서 저장하면 자체 프라이빗 AMI를 만들 수 있다.

일부 AMI는 특정 리전에서 사용할 수 없으므로 배포 시에 이를 고려해야 한다.  

### **인스턴스 유형**

애플리케이션에 딱 맞는 컴퓨팅 성능, 메모리, 스토리지 용량을 제공해 필요한 사양에 맞게 비용을 지출할 수 있도록 한다.  

- 범용
    - T3, T2, M5, M4
    - 컴퓨팅, 메모리, 네트워크 리소스를 균형있게 제공한다.
    - M5, M4는 주로 중소 규모 데이터 운영에 권장된다. EBS 가상 볼륨을 스토리지로 사용해야 하는 T2와는 달리 일부 M 인스턴스는 실제 호스트 서버에 물리적으로 연결된 내장 인스턴스 스토리지 드라이브와 함께 제공된다.
- 컴퓨팅 최적화
    - C5, C4
    - 대규모 요청을 받는 웹 서버와 고성능 머신 러닝 워크로드에 사용한다.
- 메모리 최적화
    - X1e, X1, R5, R4, z1d
    - 처리량이 많은 데이터베이스, 데이터 분석, 캐싱 작업에 유용하다.
- 가속화된 컴퓨팅
    - P3, P2, G3, F1
    - 고성능 범용 그래픽 처리 장치(GPGPU)가 제공된다.
    - 3D 시각화/렌더링, 재무 분석, 전산 유체 역학 등의 높은 워크로드에 사용된다.
- 스토리지 최적화
    - H1, I3, D2
    - 지연 시간이 짧은 대용량 인스턴스 스토리지 볼륨을 사용한다.
    - 분산 파일 시스템과 규모가 큰 데이터 처리 애플리케이션에 유용하다.

### 인스턴스 수명 주기 & 요금

- `pending`
- `running`
- `stopping`
    - 인스턴스가 중지 또는 중지-최대 절전 모드로 전환할 준비를 하고 있는 경우
        - 중지 준비 중인 경우 요금 미청구
        - 최대 절전 모드로 전환 준비 중인 경우 요금 청구
- `shutting-down`
- `terminated`
    - 예약 인스턴스인 경우 결제 옵션에 따라 기간이 종료될 때까지 요금이 청구된다.

### 인스턴스 최대 절전 모드 (Amazon EBS 지원 인스턴스에만 해당)

[인스턴스 수명 주기](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html#instance-hibernate)

인스턴스를 최대 절전 모드로 전환하면 운영 체제에 최대 절전 모드를 수행하도록 알리고, 인스턴스 메모리(RAM)의 콘텐츠를 Amazon EBS 루트 볼륨에 저장한다.  
그리고 인스턴스의 Amazon EBS 루트 볼륨과 연결된 모든 Amazon EBS 데이터 볼륨을 유지한다.  

인스턴스를 시작하면 Amazon EBS 루트 볼륨이 이전 상태로 복원되고, RAM 콘텐츠가 다시 로드된다.  
이전에 연결된 데이터 볼륨이 다시 연결되고, 인스턴스는 해당 인스턴스 ID를 유지한다.  

인스턴스를 최대 절전 모드로 전환하면 `stopping` 상태로 전환되고 나서 `stopped` 상태로 전환된다.  
최대 절전 모드로 전환하지 않고 인스턴스를 중지한 경우와 달리 최대 절전 모드 인스턴스가 `stopped` 상태이면 해당 인스턴스에 대해서는 사용 요금을 청구할 수 없지만 `stopping` 상태일 때 비용이 청구된다.  
데이터 전송에 대해 사용 요금이 부과되지는 않지만 RAM 데이터에 대한 스토리지를 포함해 모든 Amazon EBS 볼륨에 대한 스토리지 요금은 부과된다.  

### 인스턴스 구입 옵션

- 온디맨드 인스턴스
- 전용 인스턴스 (Dedicated instances)
    - 단일 고객 전용 하드웨어를 점유하는 VPC에서 사용되는 인스턴스.
- 예약 인스턴스 (Reserved instances)
    - 1년 혹은 3년 약정으로 할인된 가격에 인스턴스 이용
- 스팟 인스턴스
    - 큰 할인율로 미사용 인스턴스를 사용할 수 있게 해준다.
    - 시간을 유연하게 조정하고 중단되어도 되는 애플리케이션에서 데이터 분석, 배치 작업, 백그라운드 프로세싱 등의 용도로 사용되기에 적합하다.

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

## Amazon EBS (Elastic Block Store)

[AWS EBS 기능 - Amazon Web Services](https://aws.amazon.com/ko/ebs/features/)

EBS는 필요한 수만큼 인스턴스에 연결할 수 있으며 물리 서버의 하드 드라이브, 플래시 드라이브, USB 드라이브와 유사하게 사용된다.  

모든 EBS 볼륨은 스냅샷을 통해 복사할 수 있고, 기존 스냅샷을 다른 인스턴스에 공유해서 연결할 수 있으며, AMI에 등록할 수 있는 이미지로 변경할 수 있다.  
EBS 볼륨을 암호화해서 데이터를 보호할 수 있고, 내부에서 암호화 키를 관리하거나 AWS KMS에서 제공되는 키를 사용할 수 있다.  

### EBS 유형

현재 EBS 유형에는 SSD(Solid State Drive) 기술을 사용하는 두 가지 유형과 기존의 디스크 회전 구동형 HDD(Hard Disk Drive) 기술을 사용하는 두 가지 유형이 있다.  
성능은 최고 **IOPS(Input/Output Operations Per Second)**로 측정한다.  

- EBS 프로비저닝된 IOPS SSD (io2)
    - 고성능 I/O 작업이 필요할 때 사용한다.
    - 최대 64,000 IOPS 성능을 제공한다.
    - GB 당 주어지는 IOPS는 500:1이다.
- EBS 프로비저닝된 IOPS SSD (io1)
    - 최대 64,000 IOPS 성능을 제공한다.
    - GB 당 주어지는 IOPS는 50:1이다.
- EBS 범용 SSD (gp2)
    - 일반 서버 워크로드에는 이론적으로 짧은 지연 시간 성능을 제공하는 범용 SSD가 적합하다.
    - 최대 16,000 IOPS 성능을 제공한다.
    - GB 당 주어지는 IOPS는 3:1이다.
- 처리량에 최적화된 HDD (st1)
    - 로그 처리와 빅데이터 작업 등 처리량이 많은 워크로드에 적합하다.
    - 최대 500 IOPS 성능을 제공한다.
- 콜드 HDD (sc1)
    - 빈번하게 엑세스하지 않는 대용량 데이터 작업에 적합하다.
    - 250 IOPS 성능을 제공한다.

---

## VPC (Virtual Private Cloud)

[Amazon VPC란 무엇인가?](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/what-is-amazon-vpc.html)

Amazon VPC는 EC2의 네트워크 계층이며, EC2 인스턴스를 비롯한 여러 AWS 서비스에 네트워크 리소스를 담을 수 있는 가상 네트워크다.  
모든 VPC는 기본적으로 다른 모든 네트워크와 격리돼 있지만, 필요할 때는 인터넷 및 다른 VPC 등과 연결할 수 있다.  

VPC는 한 AWS 리전 안에서만 존재할 수 있으며, 한 리전에 만든 VPC는 다른 리전에서는 보이지 않는다.  
하나의 계정에 여러 VPC를 둘 수 있고 단일 리전에 여러 VPC를 만들 수 있다.  

### **VPC CIDR 블록**

VPC는 하나 이상의 연속적 IP 주소 범위로 구성되며, CIDR(Classless Inter Domain Routing) 블록으로 표시한다.  
VPC CIDR 접두사 길이는 /16부터 /28까지다.IP는 IPv4의 축약어로 유효한 접두사 길이는 /0부터 /32까지다.  
사설망(private) RFC 1918 범위는 다음과 같다.  

- 10.0.0.0 - 10.255.255.255 (10.0.0.0/8)
- 172.16.0.0 - 172.31.255.255 (172.16.0.0/12)
- 192.168.0.0 - 192.168.255.255 (192.168.0.0/16)

### **보조 CIDR 블록**

VPC를 만든 후에도 보조 CIDR 블록을 지정할 수 있다.  
보조 CIDR 블록은 기본 CIDR 주소 범위나 퍼블릭에서 라우팅이 가능한 범위 내에서 생성돼야 하고, 기본 블록 또는 다른 보조 블록과 겹치지 않아야 한다.  

### **서브넷**

서브넷은 VPC 내 논리 컨테이너로서 EC2 인스턴스를 배치하는 장소다.  
인스턴스를 서로 격리하고, 인스턴스 간 트래픽 흐름을 제어하고, 인스턴스를 기능별로 모을 수 있다.  

### **서브넷 CIDR 블록**

서브넷의 CIDR은 VPC CIDR의 일부이면서, VPC 내에서 고유해야 한다.  
AWS는 모든 서브넷에서 처음 4개 IP 주소와 마지막 IP 주소를 예약하며, 이 주소는 인스턴스에 할당할 수 없다.  

VPC는 보조 CIDR을 가질 수 있지만, 서브넷에는 하나의 CIDR만 있다.  
VPC에 기본 CIDR과 보조 CIDR이 있는 경우, 그 중 하나에서 서브넷 CIDR을 생성할 수 있다.  

### **가용 영역**

서브넷은 하나의 가용 영역(AZ, Availabiliy Zone) 내에서만 존재할 수 있다.  
AWS 리전의 가용 영역들은 서로 프라이빗으로 연결돼 있으며, 한 가용 영역에 장애가 발생하더라도 다른 영역에 장애의 영향이 미치지 않도록 설계됐다.  

서로 다른 가용 영역에 서브넷을 하나씩 만든 다음 인스턴스를 각각 서브넷에 분산해서 만들면, 애플리케이션은 복원성을 확보할 수 있다.  

### **탄력적 네트워크 인터페이스**

탄력적 네트워크 인터페이스(ENI, Elastic Network Interface)를 사용하면 인스턴스가 AWS 서비스, 다른 인스턴스, 온프레미스 서버, 인터넷 등 다른 네트워크 리소스와 통신할 수 있다.  
모든 인스턴스에는 기본 ENI가 있어야 하며 이 인터페이스는 하나의 서브넷에만 연결된다.  

각 인스턴스는 기본 프라이빗 IP 주소를 가지고 있어야 하는데, 그 주소는 서브넷 CIDR에서 지정한 범위 내 주소여야 한다.  

ENI는 인스턴스와 독립적으로 존재할 수 있는데, 먼저 ENI를 생성하고 나중에 인스턴스에 연결할 수 있다.  
기존 ENI를 가져와서 기존 인스턴스에 보조 ENI로 연결할 수 있으며, 장애가 있는 인스턴스에서 ENI를 분리해 작동하고 있는 다른 인스턴스에 연결하면 트래픽을 장애 인스턴스에서 정상 인스턴스로 전환할 수 있다.  

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

### Internet Gateway (IGW)

인터넷 게이트웨이는 퍼블릭 IP 주소를 할당받은 인스턴스가 인터넷과 연결돼서 인터넷으로부터 요청을 수신하는 기능을 제공한다.  
VPC에는 하나의 게이트웨이만 연결할 수 있다.  

인터넷 게이트웨이에는 관리형 IP 주소나 네트워크 인터페이스가 없는 대신, 식별을 위해 AWS 리소스 ID가 할당된다.  
리소스 ID는 igw-로 시작하며 그 뒤에 영숫자 문자열이 온다.  

### NAT 디바이스

네트워크 주소 변환은 인터넷 게이트웨이에서 이뤄지지만, 다음 두 가지 서비스도 네트워크 주소 변환을 수행한다.  

- NAT 게이트웨이
- NAT 인스턴스

AWS는 이를 `NAT 디바이스`라고 하며, 인스턴스가 인터넷에 액세스할 수 있게 하면서 인터넷상의 호스트에서는 인스턴스에 직접 도달하지 못하게 할 때 사용한다.  
이 기능은 인스턴스가 업데이트 패치를 받거나 데이터를 업로드할 때 인터넷에 연결할 필요는 있지만, 클라이언트 요청에 응답할 필요는 없을 때 유용하다.  

### **NAT 게이트웨이**

NAT 게이트웨이는 AWS에서 관리하는 NAT 디바이스이며, 인터넷 게이트웨이처럼 하나의 NAT 게이트웨이로 어떠한 용량의 요청도 처리할 수 있다.  

NAT 게이트웨이를 생성할 때 EIP도 함께 할당해서 연결해야 하고, 퍼블릭 서브넷 한 곳에 배포해서 인터넷에 액세스할 수 있게 해야 한다.  

### **NAT 인스턴스**

NAT 인스턴스는 사전 구성된 Linux 기반 AMI를 사용하는 일반적인 EC2 인스턴스이며, 여러 면에서 NAT 게이트웨이처럼 작동하지만 NAT 인스턴스는 몇 가지 중요한 다른 점이 있다.  

NAT 게이트웨이와는 달리 NAT 인스턴스는 대역폭 요구가 증가하더라도 자동으로 확장되지 않는다.  
따라서 적절한 성능을 갖춘 인스턴스 유형을 선택히는 것이 중요하다.  
대역폭 요구가 증가하면 직접 더 큰 인스턴스 유형으로 업그레이드해야 한다.  

그리고 NAT 인스턴스에는 ENI가 있으므로 보안 그룹을 적용해야 하고 퍼블릭 IP 주소를 할당해야 한다.  

NAT 인스턴스의 한 가지 이점은 배스천 호스트(Bastion Host, 또는 점프 호스트)로 사용해서 퍼블릭 IP가 없는 인스턴스에 연결할 수 있다는 것이며, NAT 게이트웨이로는 이 작업을 수행할 수 없다.  

인스턴스나 가용 영역에 장애가 발생하면 다른 NAT 인스턴스로 확장하는 정도로는 간단히 대처할 수 없는데 이는 기본 라우팅에 여러 NAT 인스턴스를 대상으로 경로를 지정할 수 없기 때문이다.  

---

## Amazon S3

[Amazon Simple Storage Service(S3) - 클라우드 스토리지 - AWS](https://aws.amazon.com/ko/s3/faqs/)

Amazon 스토리지 서비스.  

### 알림

Amazon S3는 객체 생성/복제/제거/복원/손실 등의 알림을 Amazon SNS, Amazon SQS, AWS Lambda로 보낼 수 있다.  

### 데이터 크기

일반 single PUT 요청(데이터 저장)의 데이터 한도는 `5GB` 이다.  
그 이상의 파일은 Multipart Upload를 사용해야 한다.  

### Lifecycle Configuration

S3에는 Lifecycle Configuration이 있다.  

- Transition Actions : 특정 시간이 지나면 하나의 스토리지에 있는 객체를 다른 스토리지로 옮기도록 설정할 수 있다.
    - Standard-IA에 있던 데이터를 30일이 지나면 훨씬 저렴한 Glacier로 옮기도록 설정할 수 있다.
- Expiration Actions : 특정 시간이 지나면 객체를 삭제하도록 설정할 수 있다.

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

## Amazon API Gateway

[Amazon API Gateway란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/apigateway/latest/developerguide/welcome.html)

컨테이너식 서버리스 워크로드, 웹 애플리케이션을 지원하는 완전 관리형 API 관리 서비스.  
트래픽 관리, CORS 지원, 권한 부여 및 액세스 제어, 제한, 모니터링 및 API 버전 관리 작업.  
애플리케이션이 뒤쪽 백엔스 서비스에 액세스할 수 있는 `정문` 역할을 한다.  

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

## AWS DynamoDB

[Amazon DynamoDB란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/Introduction.html)

완전 관리형 NoSQL 데이터베이스.  

### **Amazon DynamoDB Accelerator (DAX)**

DAX는 완전 관리형 In-Memory Read Performance를 향상시켜주는 인메모리 캐시 서비스이다.  

---

## AWS CloudFormation

[AWS CloudFormation이란 무엇인가요?](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/Welcome.html)

AWS 리소스들을 모델링하고 설정하여 해당 리소스의 프로비저닝과 구성을 담당해주는 서비스.  

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

## AWS IoT Core

[AWS IoT(이)란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/iot/latest/developerguide/what-is-aws-iot.html)

여러 장비들을 쉽고 안전하게 클라우드 애플리케이션 및 다른 디바이스와 상호 작용할 수 있게 해주는 관리형 클라우드 서비스.  

## AWS OpsWorks

[AWS OpsWorks란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/opsworks/latest/userguide/welcome.html)

Puppet 또는 Chef를 사용하여 클라우드 엔터프라이즈에서 애플리케이션을 구성하고 운영하도록 지원하는 구성 관리 서비스.  
Chef Configuration Management : 일종의 설정 자동화 툴.  

## AWS Snowball

[AWS Snowball | 안전한 엣지 컴퓨팅 및 오프라인 데이터 전송 | Amazon Web Services](https://aws.amazon.com/ko/snowball/?whats-new-cards.sort-by=item.additionalFields.postDateTime&whats-new-cards.sort-order=desc)

페타바이트 규모의 데이터를 오프라인 전송 디바이스를 이용해 블록 스토리지나 S3로 전송할 수 있는 서비스.  

## AWS Import/Export

[Upload Large Amounts of Data with AWS Import/Export](https://docs.aws.amazon.com/ko_kr/emr/latest/ManagementGuide/emr-plan-input-import-export.html)

AWS Snowball과 비슷한 마이그레이션 툴.  
이동식 스토리지 디바이스를 AWS에 보내면 AWS에서 전용 고속 인터넷으로 스토리지에 데이터를 전송한다.  

## AWS SSO (Single Sign-On)

[AWS Single Sign-On | 클라우드 SSO 서비스 | AWS](https://aws.amazon.com/ko/single-sign-on/)

모든 AWS Organizations 계정에 대한 액세스 권한을 한 곳에서 관리할 수 있는 서비스.  

## AWS Cognito

[Cognito | 계정 동기화 | Amazon Web Services](https://aws.amazon.com/ko/cognito/)

사용자 인증 관리, 소셜 서비스 자격 증명 연동 등을 지원하는 인증 서비스.
