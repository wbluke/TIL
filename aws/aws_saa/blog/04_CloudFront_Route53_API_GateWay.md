- 블로그 업로드 : https://wbluke.tistory.com/57

---

## AWS SAA

---

[SAA-C02 자격증 합격 후기](https://wbluke.tistory.com/53)에 이어 공부했던 내용들을 간단하게 다시 정리하고 있습니다.  

네 번째로 트래픽에 대한 캐싱, 분배 제어 등의 기능을 할 수 있는 CloudFront, Route 53, API Gateway에 대해서 간단하게 알아보겠습니다!  

## Amazon CloudFront

---

[사진]  

[CloutFront](https://docs.aws.amazon.com/ko_kr/AmazonCloudFront/latest/DeveloperGuide/Introduction.html)는 HTML, CSS, JS 및 이미지 파일과 같은 정적 및 동적 웹 콘텐츠를 사용자에게 더 빨리 배포하도록 지원하는 CDN(Content delivery network) 웹 서비스입니다.  
보통 S3와의 조합으로 많이 사용하는데요, S3에 정적 오리진 데이터를 넣어놓고 CloudFront를 연결한 후 글로벌한 CloudFront 엣지 로케이션에 배포해서 각 지역에 맞는 빠른 액세스 전략을 가져갈 수 있습니다.  
각 지역에서는 오리진 데이터가 있는 S3까지 오지 않아도, 끝단인 CloudFront에 미리 캐싱되어 있는 데이터를 빠르게 받아볼 수 있기 때문에 사용자 지연 시간을 처리하기 위해 많이 사용하는 전략 중 하나입니다.  

### Signed URL & Signed Cookies

CloudFront를 통해 특정 경로, 인증를 통해서만 private한 데이터에 액세스할 수 있도록 권한을 제어할 수 있는데요.  
아래 두 가지 방법이 많이 쓰이므로, 차이를 잘 알아두는 것이 좋습니다.  

- Signed URL
    - RTMP(Real Time Message Protocol)을 지원합니다.
    - 개별 파일마다 제한 조건을 둘 수 있습니다.
- Signed Cookies
    - RTMP(Real Time Message Protocol)을 지원하지 않습니다.
    - 여러 파일들에 한꺼번에 제한 조건을 걸 수 있습니다.
    - 현재 제공되고 있는 URL을 변경하지 않아도 됩니다.

### Origin Access Identity (OAI)

Amazon S3 버킷을 CloudFront 배포의 오리진으로 처음 설정하면 모든 사용자에게 버킷의 파일에 대한 권한을 부여하게 됩니다.  
이렇게 되면 누구나 CloudFront URL 혹은 S3 URL 두 경로 모두로 파일에 접근할 수 있게 됩니다.  

이 상황에서, CloudFront의 Signed URL 또는 Signed Cookies를 사용하여 Amazon S3 버킷의 파일에 대한 액세스를 제한하는 경우에는, Amazon S3 URL로 Amazon S3 파일에 직접 액세스하는 사용자도 차단하는 것이 좋습니다.  
서명된 URL, 서명된 쿠키와 관계 없이 S3 URL을 가지고 있다면 해당 파일에 바로 접근할 수 있기 때문입니다.  

따라서 OAI라는 것을 만들어 다음과 같은 작업을 수행해야 합니다.  

- 특별한 CloudFront 사용자인 OAI를 만들고 CloudFront 배포와 연결합니다.
- OAI만 읽기 권한을 가지도록 Amazon S3 버킷 권한을 변경합니다.
    - S3 URL로 파일을 읽을 수 없게 됩니다.

### Lambda@Edge

Lambda@Edge를 사용하면 서버를 프로비저닝하거나 관리하지 않고 글로벌 AWS 엣지 로케이션에서 코드를 실행할 수 있으므로 네트워크 지연 시간을 최소화하여 최종 사용자에게 바로 응답할 수 있습니다.  
Node.js나 Python 코드를 AWS Lambda에 업로드하고 Amazon CloudFront 요청에 대한 응답으로 함수가 트리거되도록 구성하면 됩니다.  

## Amazon Route 53

---

[사진]  

[Route 53](https://docs.aws.amazon.com/ko_kr/Route53/latest/DeveloperGuide/Welcome.html)은 가용성과 확장성이 우수한 DNS 웹 서비스입니다.  
다음과 같은 기능을 제공합니다.  

- 도메인 이름을 등록할 수 있습니다.
- 인터넷 트래픽을 도메인의 리소스로 라우팅합니다.
    - 도메인 혹은 하위 도메인 요청에 대해 브라우저를 웹 사이트 또는 웹 애플리케이션에 연결하도록 돕습니다.
- 리소스의 상태 확인
    - 웹 서버 같은 리소스로 자동화된 요청을 보내어 접근 및 사용이 가능하고 정상 작동 중인지 확인합니다.
    - 리소스 사용 불가 시 알림을 수신하고 다른 곳으로 트래픽을 라우팅할 수 있습니다.

### 라우팅 정책

Route 53에서 다양한 라우팅 정책을 설정할 수 있는데요, 각 정책의 특징을 잘 구분해서 알아두면 좋습니다.  

- 단순 라우팅 정책
    - 도메인에 대해 특정 기능을 수행하는 하나의 리소스만 있는 경우(예: example.com 웹 사이트의 콘텐츠를 제공하는 하나의 웹 서버)에 사용합니다.
- 장애 조치 라우팅 정책 (Failover routing policy)
    - 액티브-패시브 장애 조치를 구성하려는 경우에 사용합니다.
- 지리 위치 라우팅 정책 (Geolocation routing policy)
    - 사용자의 위치에 기반하여 트래픽 라우팅하려는 경우에 사용합니다.
- 지리 근접 라우팅 정책 (Geoproximity routing policy)
    - 리소스의 위치를 기반으로 트래픽을 라우팅하고 필요에 따라 한 위치의 리소스에서 다른 위치의 리소스로 트래픽을 보내려는 경우에 사용합니다.
- 지연 시간 라우팅 정책 (Latency routing policy)
    - 여러 AWS 리전에 리소스가 있고 최상의 지연 시간을 제공하는 리전으로 트래픽을 라우팅하려는 경우에 사용합니다.
- 다중 응답 라우팅 정책 (Multivalue answer routing policy)
    - Route 53이 DNS 쿼리에 무작위로 선택된 최대 8개의 정상 레코드로 응답하게 하려는 경우에 사용합니다.
- 가중치 기반 라우팅 정책 (Weighted routing policy)
    - 사용자가 지정하는 비율에 따라 여러 리소스로 트래픽을 라우팅하려는 경우에 사용합니다.

### Active-Active Failover, Active-Passive Failover

액티브-액티브 장애 조치는 모든 리소스를 대부분의 시간 동안 사용 가능하도록 하는 설정입니다.  
액티브-패시브 장애 조치는 기본 리소스 그룹이 대부분의 시간 동안 사용 가능하도록 하고 **보조 리소스 그룹은 대기 상태**에 놓은 설정입니다.  
다음과 같은 구성이 가능합니다.  

- 하나의 기본 및 보조 리소스를 사용한 액티브-패시브 장애 조치 구성
- 여러 개의 기본 및 보조 리소스를 사용한 액티브-패시브 장애 조치 구성
- 가중치 레코드를 사용하여 액티브-액티브, 액티브-패시브 장애 조치 구성
    - 레코드 별로 가중치를 설정하여 DNS 쿼리에 응답합니다.
    - 가중치가 0인 레코드는 0이 아닌 가중치의 레코드가 비정상일 경우에만 쿼리에 응답합니다.

## Amazon API Gateway

---

[Amazon API Gateway](https://docs.aws.amazon.com/ko_kr/apigateway/latest/developerguide/welcome.html)는 컨테이너식 서버리스 워크로드, 웹 애플리케이션을 지원하는 완전 관리형 API 관리 서비스입니다.  
트래픽 관리, CORS 지원, 권한 부여 및 액세스 제어, 제한, 모니터링 및 API 버전 관리 작업 등의 기능을 제공합니다.  

애플리케이션이 뒤쪽 백엔스 서비스에 액세스할 수 있는 `정문` 역할을 합니다.  
주로 Lambda를 연결하여 안정적인 트래픽 관리 및 제어를 담당하도록 구성합니다.
