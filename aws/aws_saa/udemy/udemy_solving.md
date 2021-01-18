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

### 온디맨드 인스턴스 제한

리전별로 AWS 계정당 온디맨드 인스턴스 실행 수에는 제한이 있다.  
인스턴스 수는 인스턴스 유형과 관계 없이 vCPU 수를 기준으로 관리된다.  
리소스가 부족한 경우 AWS에 제한 증가를 요청할 수 있다.  

### 배치 그룹 (Placement Group)

- 클러스터 배치 그룹
    - 단일 가용 영역 내에 있는 인스턴스 논리적 그룹
    - 짧은 네트워크 지연 시간, 높은 네트워크 처리량을 요하는 애플리케이션에 권장
- 파티션 배치 그룹
    - 논리적 파티션을 나누고, 각 인스턴스 그룹이 서로 기본 하드웨어를 공유하지 않게 한다.
    - Hadoop, Cassandra, Kafka 등 대규모의 분산 및 복제된 워크로드에 필요
- 분산형 배치 그룹
    - 소규모의 인스턴스 그룹을 다른 기본 하드웨어로 분산하여 상호 관련 오류를 줄인다.
    - 동일한 리전의 여러 가용 영역에 적용될 수 있고, 그룹당 가용 영역별로 최대 7개의 인스턴스까지 가능

---

## Amazon EC2 Auto Scaling

[Amazon EC2 Auto Scaling(이)란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html)

EC2 Auto Scaling 서비스는 애플리케이션 장애를 방지하고, 장애가 발생했을 때 복구할 수 있는 방법을 제공한다.  
Auto Scaling은 지정한 개수의 EC2 인스턴스를 자동으로 프로비저닝해서 시작한다.  
수요가 증가하면 더 많은 인스턴스를 동적으로 추가할 수 있고, 인스턴스가 종료되거나 장애가 생기면 Auto Scaling이 자동으로 인스턴스를 교체한다.  

### **시작 구성**

`시작 구성(Launch Configuration)`은 인스턴스를 수동으로 생성할 때 사용하며, AMI, 인스턴스 유형, SSH 키 페어, 보안 그룹, 인스턴스 프로필, 블록 장치 연결, EBS 최적화 여부, 배치 테넌시, 사용자 데이터 등의 구성 파라미터를 지정한다.  
시작 구성은 인스턴스를 수동으로 프로비저닝할 때 입력하는 정보와 같은 내용을 담고 있는 형식 문서이다.  

시작 구성은 기존 EC2 인스턴스에서 만들 수 있고, Auto Scaling이 인스턴스에서 설정을 복사한 뒤, 필요에 따라 그 설정을 변경해서 만들 수 있으며, 아예 백지 상태에서 처음부터 만들 수도 있다.  

시작 구성은 Auto Scaling에서만 사용되므로, 인스턴스를 직접 시작할 때 시작 구성을 활용할 수 없다.  
시작 구성을 만들고 나면 수정할 수 없으므로, 설정의 일부 내용을 수정하려면 새롭게 시작 구성을 작성해야 한다.  

### **시작 템플릿**

`시작 템플릿(Launch Templates)`은 인스턴스를 수동으로 프로비저닝할 때 입력하는 정보를 설정 작업에 사용한다는 점에서 시작 구성과 유사하지만, 기존의 시작 구성보다 다양한 용도로 활용할 수 있다.  

시작 템플릿을 생성하고 난 후 수정할 수 있으며, 기존 템플릿과 수정 템플릿을 버전 별로 보관한다.  
AWS에 모든 버전을 보관하고 있다가 필요에 따라 앞 버전과 뒤 버전의 템플릿을 선택해서 사용할 수 있다.  

### **Auto Scaling Group**

Auto Scaling 그룹은 Auto Scaling이 관리하는 EC2 인스턴스의 그룹이며, Auto Scaling 그룹을 만들 때는 미리 만들어 놓은 시작 구성이나 시작 템플릿을 지정한다.  
Auto Scaling이 프로비저닝하고 유지해야 하는 가동 인스턴스의 숫자도 지정하고, Auto Scaling 그룹의 최소 및 최대 숫자도 지정하며, 유지해야 할 인스턴스의 목표 숫자도 선택적으로 설정할 수 있다.  

- 최소
    - 정상 인스턴스 수가 최솟값보다 적어지지 않도록 한다.
    - 0으로 설정하면 Auto Scaling은 인스턴스를 만들지 않고 그룹 내에서 가동 중인 모든 인스턴스를 종료한다.
- 최대
    - 정상 인스턴스 수가 최댓값을 넘지 않도록 한다.
- 목표 용량
    - 최솟값과 최댓값 사이에서 선택하며 반드시 설정해야 하는 것은 아니다.
    - 목표 용량을 지정하지 않으면 Auto Scaling은 최소 인스턴스 수로 시작하고, 목표 용량을 지정하면 Auto Scaling은 지정된 용량을 유지하기 위해 인스턴스를 추가하거나 종료한다.
    - 웹 콘솔에서는 목표 용량을 그룹 크기라고도 표시하기도 한다.

### **Application Load Balancer 대상 그룹 지정**

Application Load Balancer를 사용해 Auto Scaling 그룹에 있는 인스턴스로 트래픽을 분산하려면 Auto Scaling 그룹을 만들 때 ALB 대상 그룹의 이름을 넣는다.  
그러면 Auto Scaling이 새 인스턴스를 만들 때마다 자동으로 인스턴스를 ALB 대상 그룹에 추가한다.  

### **애플리케이션 인스턴스 상태 검사**

기본적으로 Auto Scaling은 EC2 상태 검사를 기반으로 인스턴스의 상태를 판정한다.  

ALB를 사용해 트래픽을 인스턴스로 라우팅할 때 로드 밸런서 대상 그룹에 상태 검사를 구성할 수 있다.  
대상 그룹 상태 검사는 200에서 499 범위의 HTTP 응답 코드를 검사한다.  

ALB 상태 검사에서 인스턴스가 비정상으로 확인되면 그 인스턴스로 사용자의 트래픽이 전달되지 않도록 하고, Auto Scaling은 해당 인스턴스를 제거하고 대체 인스턴스를 만들어서 로드 밸런서의 대상 그룹에 추가해 새 인스턴스로 트래픽이 전달되도록 한다.  

---

## Amazon ELB (Elastic Load Balancing)

[웹 서버 로드 밸런싱 | 서버 로드 밸런싱 | Amazon Web Services](https://aws.amazon.com/ko/elasticloadbalancing/)

모든 EC2 인스턴스에 들어오는 애플리케이션 트래픽을 분산시키는 서비스.  
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

### SSD VS. HDD

- SSD
    - small, random한 작업량(workload)
    - transactional한 작업 (OLTP)
    - IOPS 퍼포먼스가 필요한 비즈니스 애플리케이션, 데이터베이스 작업
    - 비용이 비싸다.
    - 주요 속성은 IOPS
- HDD
    - large, sequential한 작업량(workload)
    - large streaming 작업
    - 빅데이터, 데이터 웨어하우스, Log processing
    - 비용이 싸다.
    - 주요 속성은 처리량(Throughput)

### 최대 64,000 IOPS를 내기 위해

Provisioned IOPS SSD EBS volume (io1)를 사용하면서 `Nitro-based EC2`를 사용해야 한다.  
다른 EC2 인스턴스 유형에서는 Provisioned IOPS SSD EBS를 써도 최대 32,000 IOPS 밖에 나오지 않는다.  

### 암호화

EBS 데이터 볼륨, 부팅 볼륨, 스냅샷에 대한 암호화를 제공한다.  
Amazon 관리 키 또는 AWS KMS를 사용하여 생성된 키로 데이터를 암호화한다.  

SSE(Server Side Encryption)는 S3의 옵션이지 EBS의 옵션은 아니다.  

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

- 각 서브넷은 단일 가용 영역과 매핑된다.
- 생성한 모든 서브넷은 VPC의 메인 라우팅 테이블에 자동으로 매핑된다.

### **서브넷 CIDR 블록**

서브넷의 CIDR은 VPC CIDR의 일부이면서, VPC 내에서 고유해야 한다.  
AWS는 모든 서브넷에서 처음 4개 IP 주소와 마지막 IP 주소를 예약하며, 이 주소는 인스턴스에 할당할 수 없다.  

VPC는 보조 CIDR을 가질 수 있지만, 서브넷에는 하나의 CIDR만 있다.  
VPC에 기본 CIDR과 보조 CIDR이 있는 경우, 그 중 하나에서 서브넷 CIDR을 생성할 수 있다.  

### Default 서브넷

기본 서브넷은 메인 라우팅 테이블이 인터넷 게이트웨이로 서브넷의 요청을 보내기 때문에 기본적으로 public 서브넷이다.  
인터넷 게이트웨이로 향하는 기본 destination인 `0.0.0.0/0` 라우팅을 제거하고 private 서브넷으로 만들 수도 있다.  

기본 서브넷에서 시작한 EC2 인스턴스는 public IPv4 주소와 private IPv4 주소를 모두 받을 수 있다.  
반대로 기본 VPC 안의 일반 서브넷에서 시작한 인스턴스는 public 주소를 받지 못한다.  
단, 일반 서브넷에서 인스턴스 시작 마법사로 인스턴스를 생성하면 public 주소 지정 속성이 예외적으로 true이다.   

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

ENI는 인스턴스가 stopped 상태가 되더라도 분리되지 않는다.  

### VPC Peering Connection

서로 다른 두 VPC 간에 트래픽을 라우팅할 수 있도록 하는 네트워킹 연결.  
VPC 피어링 연결은 원활한 데이터 전송에 도움이 된다.  
예를 들어 AWS 계정이 두 개 이상인 경우 이들 계정을 대상으로 VPC를 피어링하여 파일 공유 네트워크를 만들 수 있다.  
VPC 피어링 연결을 사용하여 다른 VPC가 사용자의 VPC 중 하나에 있는 리소스에 액세스하도록 허용할 수도 있다.  

[지원되지 않는 VPC 피어링 구성](https://docs.aws.amazon.com/ko_kr/vpc/latest/peering/invalid-peering-configurations.html)

지원되지 않는 피어링 구성

- 겹치는 CIDR 블록
    - 중첩되는 CIDR 블록이 있으면 VPC 피어링 연결을 만들 수 없다.
- 전이적 피어링 (transitive peering)
    - A-B-C 로 연결되어 있을 때, VPC B를 통해 VPC A에서 VPC C로 직접 패킷을 라우팅할 수 없다.
- 게이트웨이 또는 private 연결을 통한 엣지 간 라우팅
    - A-B 연결에서 B에 인터넷 게이트웨이를 통핸 인터넷 연결, AWS Direct Connect 연결, 프라이빗 서브넷에서 NAT 디바이스를 통한 인터넷 연결 등이 있는 경우, A에서 해당 외부 연결에 있는 리소스에 액세스할 수 없다.

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
- 트래픽 허용 여부를 결정할 때 **번호가 가장 낮은 규칙부터 순서대로** 규칙을 처리
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

### VPN 연결

VPN 연결 옵션을 사용하여 Amazon VPC를 원격 네트워크 및 사용자에 연결할 수 있다.  

- AWS Site-to-Site VPN
    - VPC와 원격 네트워크 사이에 IPsec (IP Security) VPN 연결을 생성할 수 있다.
- AWS Client VPN
    - AWS 리소스 또는 온프레미스 네트워크에 안전하게 액세스할 수 있도록 하는 관리형 클라이언트 기반 VPN 서비스.
    - 사용자가 연결할 수 있는 엔트포인트를 구성하여 보안 TLS VPN 세션을 설정할 수 있다.
- AWS VPN CloudHub
    - 원격 네트워크가 두 개 이상인 경우(지사 사무실이 여러 곳인 경우), 가상 프라이빗 게이트웨이를 통해 AWS Site-to-Site VPN 연결을 생성할 수 있다.
- 타사 소프트웨어 VPN 어플라이언스
    - 타사 소프트웨어 VPN 어플라이언스를 실행 중인 VPC에서 EC2 인스턴스를 사용하여 VPN 연결을 생성할 수 있다.

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

아카이브를 가져올 때 다음 중 한 가지를 지정하여 가져올 수 있다.  

- 표준
    - 3~5 시간 안에 검색
- 벌크
    - 보통 5~12시간 정도로 대용량 데이터를 가져온다. 가장 저렴한 옵션이다.
- 신속 (Expedited Retrievals)
    - 몇 분 내로 신속하게 검색해야 하는 경우
    - 프로비저닝된 용량(Provisioned capacity)을 구매해야 모든 상황에 대해 신속 검색을 수행할 수 있다. (구매 안해도 신속 검색은 가능)
    - 각 용량 단위로 5분마다 신속 검색 3회를 수행할 수 있고, 최대 150MB/s의 검색 처리량이 제공된다.

### Amazon S3 Glacier Deep Archive

Amazon S3 Glacier는 장기 보관용이지만 신속 검색을 사용하여 몇 분 안에 데이터에 접근할 수 있다.  
Amazon S3 Glacier Deep Archive는 장기 보관이면서 데이터에 거의 액세스하지 않는 경우에 훨씬 더 저렴하게 이용할 수 있다.  

### Amazon S3 Transfer Acceleration

거리가 먼 클라이언트와 Amazon S3 버킷 간에 파일을 빠르고, 쉽고, 안전하게 전송할 수 있게 해준다.  
전 세계적으로 분산된 Amazon CloudFront의 AWS 엣지 로케이션을 활용한다.  

### **암호화**

- 서버 측 암호화
    - 데이터 객체를 암호화해서 디스크에 저장하는 작업과 데이터를 가져올 때 적합한 인증으로 복호화하는 작업이 이루어진다.
    - 세 가지 종류의 서버 측 암호화가 있다.
        - Amazon S3가 관리하는 암호화 키(SSE-S3)를 사용하면 AWS가 자체 엔터프라이즈 표준 키를 사용해 암호화와 복호화 프로세스의 모든 단계를 관리한다.
        - AWS KMS-관리형 키를 사용하는 서버 측 암호화(SSE-KMS)를 사용하면 SSE-S3 기능에 더해 완벽한 키 사용 추적과 봉투 키를 사용할 수 있다.
        - 고객 제공 암호화 키에 의한 서버 측 암호화(SSE-C)는 고객이 S3에 제공한 자체 키로 객체를 암호화한다.
- 클라이언트 측 암호화
    - AWS KMS-관리형 고객 마스터 키를 사용하여 업로드 전에 고유 키로 객체를 암호화한다.
    - S3 암호화 클라이언트에서 제공된 클라이언트 측 마스터 키를 사용할 수도 있다.
    - 보통은 서버 측 암호화를 사용하지만, 회사 조직의 규정 상 암호화 키의 모든 권한을 가지고 있어야 하는 경우 클라이언트 암호화를 사용해야 한다.

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

### Origin Access Identity (OAI)

Amazon S3 버킷을 CloudFront 배포의 오리진으로 처음 설정하면 모든 사용자에게 버킷의 파일에 대한 권한을 부여하게 된다.  
이렇게 되면 누구나 CloudFront URL 혹은 S3 URL로 파일에 접근할 수 있게 된다.  

CloudFront 서명된 URL 또는 서명된 쿠키를 사용하여 Amazon S3 버킷의 파일에 대한 액세스를 제한하는 경우에는 Amazon S3 URL로 Amazon S3 파일에 액세스하는 사용자도 차단하는 것이 좋다.  
서명된 URL, 서명된 쿠키와 관계 없이 S3 URL이 있다면 해당 파일에 접근할 수 있기 때문이다.  

따라서 다음과 같은 작업을 수행해야 한다.  

- 특별한 CloudFront 사용자인 OAI를 만들고 배포와 연결한다.
- OAI만 읽기 권한을 가지도록 Amazon S3 버킷 권한을 변경한다.
    - S3 URL로 파일을 읽을 수 없다.

---

## Amazon Route 53

[Amazon Route 53(이)란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/Route53/latest/DeveloperGuide/Welcome.html)

가용성과 확장성이 우수한 DNS 웹 서비스.  

- 도메인 이름 등록
- 인터넷 트래픽을 도메인의 리소스로 라우팅
    - 도메인 혹은 하위 도메인 요청에 대해 브라우저를 웹 사이트 또는 웹 애플리케이션에 연결하도록 돕는다.
- 리소스의 상태 확인
    - 웹 서버 같은 리소스로 자동화된 요청을 보내어 접근 및 사용이 가능하고 정상 작동 중인지 확인.
    - 리소스 사용 불가 시 알림을 수신하고 다른 곳으로 트래픽을 라우팅할 수 있다.

### Active-Active Failover, Active-Passive Failover

액티브-액티브 장애 조치는 모든 리소스를 대부분의 시간 동안 사용 가능하도록 하는 설정이다.  
액티브-패시브 장애 조치는 기본 리소스 그룹이 대부분의 시간 동안 사용 가능하도록 하고 보조 리소스 그룹은 대기 상태에 놓은 설정이다.  

- 하나의 기본 및 보조 리소스를 사용한 액티브-패시브 장애 조치 구성
- 여러 개의 기본 및 보조 리소스를 사용한 액티브-패시브 장애 조치 구성
- 가중치 레코드를 사용하여 액티브-액티브, 액티브-패시브 장애 조치 구성
    - 레코드 별로 가중치를 설정하여 DNS 쿼리에 응답.
    - 가중치가 0인 레코드는 0이 아닌 가중치의 레코드가 비정상일 경우에만 쿼리에 응답.

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

- 표준 대기열
    - 각 메시지의 최소 1회 전달을 보장한다.
    - 정확한 순서를 보장하지 않는다.
- FIFO 대기열
    - 정확히 1회 처리함을 보장한다.
    - 정확한 순서를 보장한다.

---

## Amazon SNS (Simple Notification Service)

[Amazon SNS란 무엇인가요?](https://docs.aws.amazon.com/ko_kr/sns/latest/dg/welcome.html)

Pub/Sub 구조에서 사용되는 메시지 전송 관리형 서비스.  

SQS가 `폴링` 모델인 반면에 SNS는 다수의 구독자에게 `푸시` 매커니즘으로 메시지를 보낸다.  

---

## Amazon MQ

[Amazon MQ란 무엇인가요?](https://docs.aws.amazon.com/ko_kr/amazon-mq/latest/developer-guide/welcome.html)

관리형 메시지 브로커 서비스.  
SQS, SNS와 다른 점은 다음과 같다.  
온프레미스 환경에서 사용하던 코드를 클라우드 환경으로 그대로 옮기고 싶거나, MQTT, STOMP 등의 다양한 프로토콜 지원이 필요한 경우에 사용한다.  
쉽게 말해 널리 사용되는 다양한 기존 메시지 브로커와 호환되는 서비스이다.  

---

## Amazon RDS

[Amazon Relational Database Service(Amazon RDS)란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/UserGuide/Welcome.html)

### 고가용성 다중 AZ 배포

[Amazon RDS를 위한 고가용성(다중 AZ)](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)

다중 AZ 배포에서 Amazon RDS는 서로 다른 가용 영역에 동기식 예비 복제본(Synchronous Standby)을 프로비저닝하고 유지한다.  
다만, 고가용성 기능은 읽기 전용 시나리오의 솔루션과 다르므로, 예비 복제본을 사용하여 읽기 트래픽을 지원할 수는 없다.  

### Read Replica

Amazon RDS 읽기 전용 복제본은 RDS 인스턴스의 성능과 내구성을 높여준다.  
읽기 전용 복제본을 사용하면 손쉽게 단일 DB 인스턴스를 탄력적으로 확장하여 읽기 중심의 데이터베이스 워크로드를 처리할 수 있다.  
필요한 경우 읽기 전용 복제본은 독립 실행형 DB 인스턴스로 승격될 수 있다.  

- 다중 AZ 배포
    - 고가용성이 주요 목적
    - 동기식 복제. (Aurora는 비동기식 복제)
- 다중 리전 배포
    - 재해 복구 및 로컬 성능이 주요 목적
    - 비동기식 복제
- 읽기 전용 복제본
    - 확장성이 주요 목적
    - 비동기식 복제

---

## Amazon Aurora

[Amazon Aurora이란 무엇인가요?](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/AuroraUserGuide/CHAP_AuroraOverview.html)

MySQL 및 PostgreSQL과 호환되는 완전 관리형 관계형 데이터베이스 엔진.  

### instance cluster + endpoint

Amazon Aurora 는 전형적으로 단일 인스턴스가 아닌 인스턴스 군을 포함한다.  
각각의 커넥션은 특정한 DB 인스턴스에 의해 관리된다.  
Aurora cluster에 접근하면, 호스트 이름과 포트번호로 endpoint라 불리는 특정 핸들러를 지정받을 수 있다.  

Aurora에서는 각각의 인스턴스 군이 서로 다른 역할을 가져갈 수 있다.  
endpoint를 사용하면 각각의 커넥션을 역할에 맞는 인스턴스 군으로 매핑할 수 있다.  

- Cluster endpoint
    - writer endpoint이다. primary DB와 연결된다.
- Reader endpoint
    - read-only 커넥션에 대한 load-balancing을 제공한다.

### Amazon Aurora Parallel Query (병렬 쿼리)

[Amazon Aurora MySQL용 Parallel Query 처리](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/AuroraUserGuide/aurora-mysql-parallel-query.html)

Amazon Aurora 병렬 쿼리는 데이터를 별도의 시스템으로 복사할 필요 없이 현재 데이터에 대한 `분석 쿼리`를 더 빠르게 제공하는 기능.  
핵심 트랜잭션 워크로드의 처리량을 높게 유지하면서 쿼리 속도를 최대 100배로 높일 수 있다.  

일부 데이터베이스는 하나 또는 몇 개 서버의 CPU에 걸쳐 쿼리 처리를 병렬화할 수 있지만, 병렬 쿼리는 Aurora의 고유한 아키텍처를 활용하여 Aurora 스토리지 계층에 있는 수천 개의 CPU 전체로 쿼리 처리를 `푸시 다운`하여 병렬화한다.  
병렬 쿼리는 분석 쿼리 처리를 Aurora 스토리지 계층으로 오프로딩하여 트랜잭션 워크로드의 네트워크, CPU 및 버퍼 풀 경합을 줄인다.  

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

CloudFormation에 대한 사용 요금은 없고, 대신 이를 통해 생성한 AWS 리소스에 대해서만 사용한 만큼 가격을 지불하면 된다.  

---

# 기타 서비스

## Redshift

[데이터 웨어하우스 | Redshift | Amazon Web Services](https://aws.amazon.com/ko/redshift/?whats-new-cards.sort-by=item.additionalFields.postDateTime&whats-new-cards.sort-order=desc)

클라우드 데이터 웨어하우스. OLAP 환경에서 사용한다.  
열 기반 스토리지(columnar storage)도 지원한다.  

## AWS Fargate

[AWS Fargate | 서버리스 컴퓨팅 엔진 | Amazon Web Services](https://aws.amazon.com/ko/fargate/?whats-new-cards.sort-by=item.additionalFields.postDateTime&whats-new-cards.sort-order=desc&fargate-blogs.sort-by=item.additionalFields.createdDate&fargate-blogs.sort-order=desc)

컨테이너에 적합한 서버리스 컴퓨팅 엔진.  
서버를 프로비저닝하고 관리할 필요가 없어 애플리케이션 빌드에만 초점을 맞출 수 있게 도와준다.  

## AWS LightSail

[Amazon Lightsail](https://aws.amazon.com/ko/lightsail/)

EC2보다 가볍고, 단순화 된 서비스.  
버스트 기능이 있는 t2 계열의 EC2 instance라 볼 수 있다.  
LightSail의 대상은 AWS의 EC2, EBS, VPC, 그리고 Route53 같은 무수한 옵션들을 고려하고 싶지 않은 “간단한 VPS를 원하는” 고객들이다.  

## Amazon Kinesis

[실시간 데이터 분석 처리 시스템 | Amazon Web Services](https://aws.amazon.com/ko/kinesis/)

실시간 비디오, 실시간 스트리밍 데이터를 수집, 처리, 분석할 수 있는 서비스.  

- Kinesis Video Streams
    - 머신러닝, 분석, 재생 등을 위해 비디오를 스트리밍하는 서비스.
- Kinesis Data Streams
    - 고도로 확장 가능하고 내구력 있는 실시간 데이터 스트리밍 서비스.
    - 웹 사이트 클릭스트림, 데이터베이스 이벤트 스트림, 금융 트랜잭션, 소셜 미디어 피드, IT 로그 및 위치 추적 이벤트 등
    - Amazon S3, AWS Lambda에 제공할 수 있다.
    - 기본적으로는 데이터를 24시간 저장하고, 최대 168시간까지 저장할 수 있다.
- Kinesis Data Firehose
    - 스트리밍 데이터를 데이터 레이크, 데이터 스토어 및 분석 서비스에 안정적으로 로드.
    - Amazon S3, Amazon Redshift, Amazon Elasticsearch Service, 일반 HTTP 엔드포인트 및 Datadog, New Relic, MongoDB, Splunk와 같은 서비스 공급자로 전송
- Kinesis Data Analytics
    - 데이터 스트림 처리를 위한 오픈 소스 프레임워크 서버리스 엔진인 Apache Flink를 통해 실시간으로 스트리밍 데이터를 변환 및 분석.

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

## AWS Shield

[AWS Shield - Amazon Web Services(AWS)](https://aws.amazon.com/ko/shield/?whats-new-cards.sort-by=item.additionalFields.postDateTime&whats-new-cards.sort-order=desc)

DDoS 공격으로부터 VPC를 보호하는 서비스.  

## AWS WAF (Web Application Firewall)

[AWS WAF - 웹 애플리케이션 방화벽 - Amazon Web Services(AWS)](https://aws.amazon.com/ko/waf/)

가용성에 영향을 주거나, 보안을 위협하거나, 리소스를 과도하게 사용하는 일반적인 웹 공격으로부터 웹 애플리케이션이나 API를 보호하는 웹 애플리케이션 방화벽.  

## AWS App Mesh

[AWS App Mesh - 모든 서비스를 위한 애플리케이션 수준의 네트워킹 - Amazon Web Services](https://aws.amazon.com/ko/app-mesh/)

애플리케이션 레벨의 네트워킹을 통해 모든 서비스에 대한 통신 및 모니터링을 구축할 수 있는 서비스.  

## AWS Cloud Map

[Cloud Map - 클라우드 리소스를 위한 서비스 검색](https://aws.amazon.com/ko/cloud-map/)

클라우드 리소스 검색 서비스.  
애플리케이션 리소스에 사용자 지정 이름을 정의하고 동적으로 변화하는 리소스의 업데이트 위치를 유지 관리하여 애플리케이션 가용성을 향상시킨다.  

## AWS AppSync

[AWS AppSync | 관리형 GraphQL API | Amazon Web Services](https://aws.amazon.com/ko/appsync/)

GrappQL API 개발을 용이하게 하는 완전 관리형 서비스.  
AWS DynamoDB, Lambda와 결합하여 사용할 수 있으며 성능을 위한 캐싱, 수백만 개 클라이언트의 실시간 데이터 업데이트를 지원한다.  

## AWS Mobile Hub

[AWS Mobile Hub - 모바일 앱 빌드, 테스트 및 모니터링 | Amazon Web Services](https://aws.amazon.com/ko/blogs/korea/aws-mobile-hub-build-test-and-monitor-mobile-applications/)

여러 AWS 서비스를 사용하여 모바일 앱 백엔드 기능을 쉽게 배포하고 구성할 수 있게 해주는 서비스.  

## Amazon EMR

[Amazon EMR이란 무엇입니까?](https://docs.aws.amazon.com/ko_kr/emr/latest/ManagementGuide/emr-what-is-emr.html)

오픈소스를 활용한 클라우드 빅데이터 플랫폼.  
대용량 빅데이터 분석으로 기계 학습, 실시간 스트리밍, 클릭스트림 분석, 유전체학 등에 사용된다.  

## AWS EFA (Elastic Fabric Adapter)

[Elastic Fabric Adapter(EFA)](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/efa.html)

EC2 인스턴스에 연결하여 HPC(고성능 컴퓨팅) 및 기계 학습 애플리케이션 속도를 높일 수 있는 네트워크 디바이스.  

## AWS Direct Connect

[AWS Direct Connect](https://aws.amazon.com/ko/directconnect/)

온프레미스에서 AWS로 전용 네트워크 연결을 쉽게 설정할 수 있는 클라우드 서비스 솔루션.  
대역폭 사용량이 많은 워크로드에서 비용 절감 효과.
