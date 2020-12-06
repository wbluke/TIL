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

클라우드 데이터 웨어하우스. OLAP 환경에서 사용한다.  

[데이터 웨어하우스 | Redshift | Amazon Web Services](https://aws.amazon.com/ko/redshift/?whats-new-cards.sort-by=item.additionalFields.postDateTime&whats-new-cards.sort-order=desc)