- 블로그 업로드 : https://wbluke.tistory.com/56

---

## AWS SAA

---

[SAA-C02 자격증 합격 후기](https://wbluke.tistory.com/53)에 이어 공부했던 내용들을 간단하게 다시 정리하고 있습니다.  

세 번째로는 가장 활용도가 높은 여러가지 Storage Service에 대해서 알아보겠습니다.  

## Amazon EBS (Elastic Block Store)

---

[사진]  

[Amazon EBS](https://aws.amazon.com/ko/ebs/features/)는 대규모의 처리량이 필요하거나 고성능의 트랜잭션 워크로드 지원이 필요한 경우에 많이 사용되는 블록 스토리지 서비스입니다.  
필요한 수만큼 EC2 인스턴스에 연결할 수 있으며 물리 서버의 하드 드라이브, 플래시 드라이브, USB 드라이브와 유사하게 사용됩니다.  

### EBS 유형

현재 EBS 유형에는 SSD(Solid State Drive) 기술을 사용하는 유형과 기존의 디스크 회전 구동형 HDD(Hard Disk Drive) 기술을 사용하는 유형으로 두 가지가 있습니다.  
각 유형의 차이를 정확히 아는 것이 중요합니다.  

성능은 최고 **IOPS(Input/Output Operations Per Second)**로 측정합니다.  

- EBS 프로비저닝된 IOPS SSD (io2)
    - 고성능 I/O 작업이 필요할 때 사용하며, 지연 시간에 민감한 트랜잭션 워크로드를 위해 설계된 볼륨입니다.
    - 최대 64,000 IOPS 성능을 제공합니다.
    - GiB 당 주어지는 IOPS는 500:1 입니다.
- EBS 프로비저닝된 IOPS SSD (io1)
    - 최대 64,000 IOPS 성능을 제공한다.
    - GiB 당 주어지는 IOPS는 50:1 입니다.
    - 참고로 시험에서 io2와 io1의 차이를 묻는 문제는 나오지 않고 있습니다.
- EBS 범용 SSD (gp2)
    - 일반 서버 워크로드에는 이론적으로 짧은 지연 시간 성능을 제공하는 범용 SSD가 적합합니다.
    - 최대 16,000 IOPS 성능을 제공합니다.
    - GiB 당 주어지는 IOPS는 3:1 입니다.
- 처리량에 최적화된 HDD (st1)
    - 로그 처리와 빅데이터 작업 등 처리량이 많은 워크로드에 적합합니다.
    - 최대 500 IOPS 성능을 제공합니다.
- 콜드 HDD (sc1)
    - 빈번하게 엑세스하지 않는 대용량 데이터 작업에 적합합니다.
    - 250 IOPS 성능을 제공합니다.

### SSD VS. HDD

위의 유형을 둘로 나눈 SSD와 HDD의 차이를 아는 것도 중요합니다.  

- SSD
    - Small, Random한 작업(workload)
    - Transactional한 작업 (OLTP)
    - IOPS 퍼포먼스가 필요한 비즈니스 애플리케이션, 데이터베이스 작업
    - 비용이 비쌉니다.
    - 주요 속성은 IOPS 입니다.
- HDD
    - Large, Sequential한 작업(workload)
    - Large Streaming 작업
    - 빅데이터, 데이터 웨어하우스, Log processing
    - 비용이 상대적으로 쌉니다.
    - 주요 속성은 처리량(Throughput) 입니다.

### 최대 64,000 IOPS를 내기 위해

최대 성능인 64,000 IOPS를 내려면 Provisioned IOPS SSD EBS volume (io1)를 사용하면서 전용 EC2 유형인 `Nitro-based EC2`를 사용해야 합니다.  
다른 EC2 인스턴스 유형에서는 io1 EBS를 써도 최대 32,000 IOPS 까지밖에 나오지 않습니다.  

### 암호화

EBS 볼륨을 암호화해서 데이터를 보호할 수 있고, 내부에서 암호화 키를 관리하거나 AWS KMS에서 제공되는 키를 사용할 수 있습니다.  
데이터 볼륨, 부팅 볼륨, 스냅샷에 대한 암호화를 제공합니다.  

> AWS KMS (Key Management Service) : 암호화 키를 생성 및 관리할 수 있는 서비스

참고로 SSE(Server Side Encryption)는 S3의 옵션이지 EBS의 옵션은 아닙니다.  

### 스냅샷

모든 EBS 볼륨은 스냅샷을 통해 복사할 수 있고, 기존 스냅샷을 다른 인스턴스에 공유해서 연결할 수 있으며, AMI에 등록할 수 있는 이미지로 변경할 수 있습니다.  

- 루트(Root) 디바이스 역할을 하는 EBS 볼륨의 스냅샷을 생성할 때는 인스턴스를 중지한 후 스냅샷을 생성해야 합니다.
- 최대 절전 모드가 활성화된 인스턴스에서는 스냅샷을 생성할 수 없습니다.
- 볼륨의 이전 스냅샷이 `pending` 상태일 때에도 볼륨의 스냅샷을 생성할 수는 있지만 볼륨의 `pending` 스냅샷을 여러 개 생성하면 스냅샷이 완료될 때까지 볼륨 성능이 저하될 수 있습니다.

## Amazon S3

---

[사진]  

[Amazon S3](https://aws.amazon.com/ko/s3/faqs/)는 가장 친숙하고 활용도가 높은 Amazon의 대표적인 스토리지 서비스입니다.  
S3는 필요한 상황에 따라 일반 S3 Standard 클래스 말고도 다양한 유형의 스토리지를 활용할 수 있는데요, 각 스토리지 유형의 특성을 정확히 알아야 합니다.  

### S3 스탠다드-Infrequent Access(S3 스탠다드-IA)

**액세스 빈도가 낮지만 필요할 때 빠르게 액세스해야 하는 데이터**를 위한 스토리지입니다.  
장기 스토리지, 백업 및 재해 복구 파일용 데이터 스토어에 이상적입니다.  

### Amazon S3 One Zone-Infrequent Access(S3 One Zone-IA)

하나의 가용 영역 안에서만 중복 저장하여 다중 가용 영역에 데이터를 저장하는 스탠다드 IA 스토리지보다 20% 저렴하게 이용할 수 있습니다.  
스탠다드와는 다르게 가용성 영역 파괴 시 데이터가 손실됩니다.  

이런 이유로 **자주 액세스 하지 않는 데이터**를 적재할 때 사용합니다.  

### Amazon S3 Glacier

Amazon S3 Glacier는 **장기 백업**을 위한 안정적이고 비용이 매우 저렴한 스토리지 클래스입니다.  

Glacier에 저장된 아카이빙 데이터를 가져올 때 다음 중 한 가지를 지정하여 가져올 수 있습니다.  

- 표준 검색
    - 3~5 시간 안에 검색.
- 벌크 검색
    - 보통 5~12시간 정도로 대용량 데이터를 가져옵니다. 가장 저렴한 옵션.
- 신속 검색 (Expedited Retrievals)
    - 몇 분 내로 신속하게 검색해야 하는 경우
    - 프로비저닝된 용량(Provisioned capacity)을 구매해야 모든 상황에 대해 신속 검색을 수행할 수 있습니다. (구매를 안해도 신속 검색은 가능)
    - 각 용량 단위로 5분마다 신속 검색 3회를 수행할 수 있고, 최대 150MB/s의 검색 처리량이 제공됩니다.

### Amazon S3 Glacier Deep Archive

Glacier는 장기 보관용이지만 신속 검색을 사용하여 몇 분 안에 데이터에 접근할 수 있는데요.  
그와 다르게 Glacier Deep Archive는 **장기 보관이면서 데이터에 거의 액세스하지 않는 경우**에 훨씬 더 저렴하게 이용할 수 있는 스토리지 클래스입니다.  

### S3 Intelligent-Tiering

액세스가 빈번할지, 간헐적일지 모르겠을 때 사용하는 스토리지 클래스입니다.  

크게 위에서 소개한 각각의 유형을 나타낸 4가지 계층 사이에서 데이터를 이동시키면서 관리하는 스토리지인데요.  

S3 Intelligent-Tiering에 업로드하거나 이전한 객체는 처음에 Frequent Access 계층에 자동으로 저장됩니다.  
S3 Intelligent-Tiering은 액세스 패턴을 모니터링한 후 30일 연속 액세스되지 않은 객체를, Infrequent Access 계층으로 이동하는 방식으로 작동합니다.  
이 두 아카이브 액세스 계층을 하나 또는 모두 활성화하면 S3 Intelligent-Tiering이 90일 연속으로 액세스되지 않은 객체를 자동으로 Archive Access 계층으로 옮기고, 이후 180일 연속으로 액세스되지 않으면 다시 Deep Archive Access 계층으로 옮깁니다.  
이후 객체에 액세스하면 S3 Intelligent-Tiering은 객체를 Frequent Access 티어로 다시 이동시킵니다.  

- 변화하는 액세스 패턴으로 데이터의 스토리지 비용 자동 최적화
- Frequent, Infrequent, Archive 및 Deep Archive 액세스에 최적화된 4개의 액세스 티어에 객체 저장
- Frequent 및 Infrequent Access 티어는 S3 Standard와 동일한 짧은 지연 시간과 높은 처리 성능 제공
- 드물게 액세스되는 객체에 대해서는 선택적인 자동 아카이브 기능 활성화
- Archive Access 및 Deep Archive Access 티어는 Glacier 및 Glacier Deep Archive와 동일한 성능 제공

### Lifecycle Configuration

S3에는 Lifecycle Configuration(수명 주기)이 있습니다.  
일정 시간이 지나면 데이터를 다른 유형의 S3로 옮길 수 있는 옵션입니다.  

- Transition Actions : 특정 시간이 지나면 하나의 스토리지에 있는 객체를 다른 스토리지로 옮기도록 설정할 수 있습니다.
    - Standard-IA에 있던 데이터를 30일이 지나면 훨씬 저렴한 Glacier로 옮기도록 설정할 수 있습니다.
- Expiration Actions : 특정 시간이 지나면 객체를 삭제하도록 설정할 수 있습니다.

참고로 S3 Standard-IA와 S3 Standard One-Zone-IA로 전환하는 수명주기 규칙은 최소 S3 Standard 클래스에 30일 이상 보관한 객체만 가능합니다.  

### 데이터 업로드 한도

일반 single PUT 요청(데이터 저장)의 데이터 한도는 `5GB` 입니다.  
그 이상의 파일은 Multipart Upload를 사용해야 합니다.  

### **암호화**

- 서버 측 암호화
    - 데이터 객체를 암호화해서 디스크에 저장하는 작업과 데이터를 가져올 때 적합한 인증으로 복호화하는 작업이 이루어집니다.
    - 세 가지 종류의 서버 측 암호화가 있습니다.
        - Amazon S3가 관리하는 암호화 키(`SSE-S3`)를 사용하면 AWS가 자체 엔터프라이즈 표준 키를 사용해 암호화와 복호화 프로세스의 모든 단계를 관리합니다.
        - AWS KMS-관리형 키를 사용하는 서버 측 암호화(`SSE-KMS`)를 사용하면 SSE-S3 기능에 더해 완벽한 키 사용 추적과 `봉투 키`를 사용할 수 있습니다.
        - 고객 제공 암호화 키에 의한 서버 측 암호화(`SSE-C`)는 고객이 S3에 제공한 자체 키로 객체를 암호화합니다.
- 클라이언트 측 암호화
    - AWS KMS-관리형 고객 마스터 키를 사용하여 업로드 전에 고유 키로 객체를 암호화합니다.
    - S3 암호화 클라이언트에서 제공된 클라이언트 측 마스터 키를 사용할 수도 있습니다.
    - 보통은 서버 측 암호화를 사용하지만, 회사 조직의 규정 상 암호화 키의 모든 권한을 가지고 있어야 하는 경우 클라이언트 암호화를 사용해야 합니다.

> [봉투 키](https://docs.aws.amazon.com/ko_kr/kms/latest/developerguide/concepts.html#enveloping) : 데이터를 암호화 한 암호화 키를 다시 다른 키로 암호화하여 관리하는 기법

### 복제

- 교차 리전 복제 (CRR)
    - 서로 다른 리전에 객체를 복사합니다.
    - 지연 시간을 최소화합니다.
- 동일 리전 복제 (SRR)
    - 동일 리전에 객체를 복사합니다.
    - 단일 버킷에 로그 집계, 프로덕션 계정과 테스트 계정 간에 라이브 복제를 구성합니다.

### 알림

Amazon S3는 객체 생성/복제/제거/복원/손실 등의 알림을 Amazon SNS, Amazon SQS, AWS Lambda로 보낼 수 있습니다.  

### Amazon S3 Transfer Acceleration

거리가 먼 클라이언트와 Amazon S3 버킷 간에 파일을 빠르고, 쉽고, 안전하게 전송할 수 있게 해줍니다.  
전 세계적으로 분산된 Amazon CloudFront의 AWS 엣지 로케이션을 활용합니다.  

## AWS Storage Gateway

---

[AWS Storage Gateway](https://docs.aws.amazon.com/ko_kr/storagegateway/latest/userguide/WhatIsStorageGateway.html)는 온프레미스 환경을 클라우드 스토리지와 연결할 수 있는 서비스입니다.  
보통 온프레미스의 백업 데이터를 받아서 클라우드 환경의 S3로 넘기는 작업을 합니다.  
즉, 온프레미스 환경에 있던 데이터를 일정 시간이 지나면 AWS로 옮긴다는 뜻이고, AWS DataSync와는 다르게 Hybrid Cloud Storage를 구현할 때 사용합니다.  

- 백업을 클라우드로 이동
- 온프레미스 파일 공유
- AWS의 데이터에 낮은 지연 시간으로 액세스 가능

Storage Gateway는 대용량 데이터를 적재하는 데에는 적합하지 않습니다.   
보통 변경된 데이터나 압축된 데이터에만 적절합니다.  
대용량 데이터를 적재하고 싶다면 Storage Gateway 대신 AWS DataSync를 사용해야 합니다.  

`파일 게이트웨이`, `테이프 게이트웨이`, `볼륨 게이트웨이`의 3가지 게이트웨이를 제공합니다.  

### 파일 게이트웨이

**NFS(Network File System) 및 SMB(Server Message Block)** 같은 업계 표준 파일 프로토콜을 사용하여 Amazon S3에서 객체를 저장하고 검색합니다.  

### 볼륨 게이트웨이

- **캐시**된 볼륨 아키텍처
    - 캐시 볼륨으로 Amazon S3를 기본 데이터 스토리지로 사용함과 동시에, 자주 액세스하는 데이터를 스토리지 게이트웨이에 로컬 보관합니다.
- **저장**된 볼륨 아키텍처
    - 저장 볼륨을 사용하여 기본 데이터를 로컬에 저장하는 한편, 해당 데이터를 AWS에 비동기식으로 백업합니다.

### 테이프 게이트웨이

백업 데이터를 비용 효율 및 내구성이 좋은 방식으로 Glacier 또는 Glacier Deep Archive에 보관합니다.  

## AWS DataSync

---

[AWS DataSync](https://docs.aws.amazon.com/ko_kr/datasync/latest/userguide/what-is-datasync.html)는 간단하게 **엄청나게 많은 양의 데이터**를 온프레미스 환경에서 Amazon S3, Amazon EFS(Elastic File System), Windows 파일 서버를 위한 Amazon FSx 등으로 적재할 수 있는 서비스입니다.  
보통 대량의 히스토리성 콜드 데이터들을 Amazon S3 Glacier나 Amazon S3 Glacier Deep Archive에 옮길 때 사용합니다.  

## Amazon EFS (Elastic File System)

---

[Amazon EFS](https://docs.aws.amazon.com/ko_kr/efs/latest/ug/whatisefs.html)는 **완전 관리형 파일 시스템**으로, 강한 일관성과 file locking이 필요한 경우 사용합니다.  
파일 추가/삭제에 따라 자동으로 용량이 확장/축소됩니다.  

EBS가 단일 AZ에 데이터를 저장하는데 비해, EFS는 다중 AZ에 데이터를 저장하고, **다중 AZ의 EC2에서 동시 연결이 가능**합니다.  

## Amazon FSx for Windows File Server

---

[Amazon FSx](https://docs.aws.amazon.com/ko_kr/fsx/latest/WindowsGuide/what-is.html)는 **완전 관리형 Windows 파일 시스템**입니다.  
Amazon FSx 은 Windows 파일 시스템 기능과 네트워크상에서 파일 스토리지에 액세스할 수 있는 업계 표준 **SMB(Server Message Block) 프로토콜**을 기본적으로 지원합니다.
