# RestTemplate
#TIL/spring

---

Spring 3.0 부터 지원하는 HTTP 통신 템플릿이다.  

get, post, put, delete 의 HTTP 메소드를 사용하여 통신할 수 있다.  

주요 메소드는 다음과 같다.  

- getForObject(), postForObject() : get, post 요청을 보내고 파라미터로 지정한 자바 오브젝트로 응답을 받는다.
	- 기대하는 json 응답을 자바 오브젝트로 매핑해서 가져올 수 있다.
	- 필요하면 `@JsonProperty` 를 사용할 수도 있다.
- getForEntity() , postForEntity() : get, post 요청을 보내고 ResponseEntity로 응답을 받는다.
- postForLocation() : 객체 반환 대신 생성된 리소스의 URI를 응답받는다.
- put, delete : put, delete 요청을 보낸다.
- execute() : HTTP Method를 지정해서 요청을 보낼 수 있다. 파라미터로 요청과 응답에 대한 
- exchange() : HTTP Method를 지정하고, 헤더를 세팅해서 요청을 보낸다. 응답은 ResponseEntity로 받는다.

setErrorHandler 메소드를 통해 에러 상황에 대한 custom 핸들러도 만들어서 지정할 수 있다.  

또한 connection에 대한 timeout 설정도 지정할 수 있다.  
