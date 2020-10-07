# ELK 스택 기본 개념 정리
#TIL/elasticsearch

---

## ElasticSearch

### 개요

RDB는 document-context 기반으로 저장되는 반면에, ES는 text 단위로 text-document 방식으로 데이터를 저장한다.  
예를 들어 John이라는 단어가 들어있는 데이터를 찾으려면, RDB는 document를 하나씩 검색해야하는 반면에, ES는 바로 text에서 John을 찾으면 해당 키워드가 들어있는 document를 모두 확인할 수 있게 된다.  
따라서 `검색`이라는 행위에서는 RDB보다 ES가 훨씬 빠르다.  

### 용어 비교

ES와 RDB의 용어를 비교하면 다음과 같다.  

- Index : Database
- Type : Table
- Document : Row
- Field : Column
- Mapping : Schema
- GET : Select
- PUT : Update
- POST : Insert
- DELETE : Delete

### REST API로 기본 사용

또한 우분투에서 CRUD를 다음과 같이 curl로 수행할 수 있다.  

```bash
curl -XGET localhost:9200/classes/class/1
curl -XPOST localhost:9200/classes/class/1 -d '{xxx}'
curl -XPUT localhost:9200/classes/class/1 -d '{xxx}'
curl -XDELETE localhost:9200/classes/class/1
```

pretty 옵션으로 json 결과값을 예쁘게 볼 수 있다.  

```bash
curl -XGET localhost:9200/classes/class/1?pretty
```

인덱스를 생성할 때는 PUT 명령어로 수행한다.  
인덱스 생성 후 document를 생성할 때는 POST로 수행한다.  

document 생성 시 파일로 생성하려면 json 파일명에 `@`을 붙여서 수행할 수 있다.  

```bash
curl -XPOST localhost:9200/classes/class/1 -d @oneclass.json
```

### bulk

bulk insert는 다음과 같이 할 수 있다.  

```bash
curl -XPOST localhost:9200/_bulk --data-binary @bulk.json
```


### Mapping

mapping이 없다면 모든 타입을 문자열로 저장할 위험이 있다.  
각 항목에 알맞은 타입을 주기 위해 mapping을 지정할 수 있다.  
또한 타입이 정해져 있으면 aggregation이나 시각화할 때 해당 데이터들을 적절히 활용할 수 있다.  

```bash
curl -XPUT localhost:9200/classes/class/_mapping -d @classesRating_mapping.json
```

```json
{
  "class": {
    "properties": {
      "title": { "type": "string" },
      "student_count": { "type": "integer" },
      "submit_date": { "type": "date", "format": "yyyy-MM-dd" },
      "school_location": { "type": "geo_point" }
    }
  }
}
```

### search

다음과 같이 모든 document를 확인할 수 있다.  

```bash
curl -XGET localhost:9200/basketball/record/_search?pretty
```

URI 옵션이라는 것이 있다.  

```bash
curl -XGET localhost:9200/basketball/record/_search?q=points:30&pretty
```

q는 query를 의미하고, points가 30인 document를 검색한다는 의미이다.  

같은 검색조건으로 Request body를 활용하는 방법이 있다.  

```bash
curl -XGET localhost:9200/basketball/record/_search -d '
{
  "query": {
    "term": {"points": 30}
  }
}'
```

### Metric Aggregations

Aggregation은 조합을 통해 어떤 값을 구할 때 사용하는 방법이다.  
그중에서도 Metric Aggregation은 최댓값, 최솟값 등 산수를 통해 값을 도출하는 방법이다.  

```json
{
  "size": 0,
  "aggs": {
    "avg_score": {
      "avg": { 
        "field": "points" 
      }
    }
  }
}
```

평균을 구하는 aggregation 이다.  

- 결과값에 보고싶은 값만 보고 싶은 경우 size를 0으로 준다.  
- aggs는 aggregations의 축약이다.  
- avg_score은 aggs의 이름이고, 중요한 것은 평균을 구하겠다는 뜻의 avg이다.
	- max, min, sum 등으로 바꾸면 그에 맞는 값을 구할 수 있다.  
	- stats라는 명령어를 사용하면 count, min, max, avg, sum 의 값들을 전부 볼 수 있다.

### Bucket Aggregations

group by 를 생각하면 된다.  

```json
{
  "size": 0,
  "aggs": {
    "players": {
      "terms": { 
        "field": "team" 
      }
    }
  }
}
```

- terms 명령어는 지정한 필드별로 group by를 해준다.  

```json
{
  "size": 0,
  "aggs": {
    "team_stats": {
      "terms": { 
        "field": "team" 
      },
      "aggs": {
        "stats_score": {
          "stats": { 
            "field": "points" 
          }
        }
      }
    }
  }
}
```

- 중첩으로 aggs를 넣을 수 있다.
- stats 명령어는 통계를 의미한다. 따라서 이는 팀별로 통계를 볼 수 있는 aggregation이다.

---

## 키바나 (Kibana)

### 설치

키바나 설치 후 설정 파일에서 다음 두 가지를 확인해야 한다.  
설정 파일 위치는 `/etc/kibana/kibana.yml` 이다.  

- elasticsearch.url
	- 같은 서버라면 `http://localhost:9200`
- server.host
	- 같은 서버라면 `localhost`

kibana는 다음과 같이 시작하면 된다.  

```bash
sudo /usr/share/kibana/bin/kibana
```

kibana의 기본 포트는 5601이다.  

---

## Logstash

### 개요

Logstash는 input을 담당한다.  
Logstash는 여러 데이터셋을 흡수해, 자기가 원하는 형태로 데이터를 가공할 수 있고, 가공한 데이터를 ES로 넘겨줄 수 있다.  

Logstash에는 config 파일이 필요하다.  

```json
input {
  stdin { }
}
output {
  stdout { }
}
```

표준입출력을 사용한다는 의미의 config 파일이다.  
config 파일에는 다음과 같이 세 파트가 존재한다.  

- input
	- 데이터의 입력을 받을 file이나 application 설정을 준다.
- filter
	- 구분자, column, 변환 등을 설정할 수 있다.
- output
	- 표준입출력을 통한 콘솔 출력, ES 등으로 보내도록 설정할 수 있다.















































