# Real MySQL - 6. 실행 계획
#TIL/Database


## 실행 계획?

### 최적의 방법을 찾아내기 위해

수많은 데이터에서 내가 원하는 데이터를 뽑아내기 위한 방법은 정말 다양할 수 있다. 그렇기에 우리는 어떤 방법이 최적이고 최소의 비용이 소모될지 결정해야 한다.

여행 계획을 세부적으로 따져가면서 세우듯이, DBMS에서도 쿼리를 최적으로 실행하기 위해 각 테이블의 데이터가 어떤 분포로 저장되어 있는지 **통계 정보**를 참조하고, 최적의 실행계획을 수립한다. 옵티마이저가 이런 작업을 담당한다.  

MySQL에서는 `EXPLAIN`이라는 키워드로 실행 계획을 확인할 수 있다.  

> 따라서 통계 정보는 실행 계획에서 상당히 중요한데, 레코드 건수가 많지 않으면 통계 정보가 부정확하여 엉뚱한 실행 계획이 도출될 수 있음을 늘 염두에 두고 있어야 한다. 필요에 따라 `ANALYZE` 명령어로 통계정보를 강제적으로 갱신해야 할 수도 있다.  

- - - -

## 실행 계획 분석

`EXPLAIN` 명령어로 내가 실행할 쿼리의 실행계획을 보면, 테이블 형태로 메타 데이터들이 조회되는 것을 볼 수 있다. 따라서 각각의 칼럼들의 의미를 아는 것이 중요한데, 책에서는 거의 모든 내용을 소개하지만 이 글에서는 필요한 몇 가지만 짚으려 한다.

### id 칼럼

### select_type 칼럼

- SIMPLE : 
- PRIMARY : 
- UNION : 
- DEPENDENT UNION : 
- SUBQUERY : 
- DEPENDENT SUBQUERY : 
- DERIVED : 
- UNCACHEABLE SUBQUERY : 

### type 칼럼

- const
- eq_ref
- ref
- fulltext
- unique_subquery
- index_subquery
- range
- index_merge
- index
- all

### possible_keys 칼럼

### key 칼럼

### key_len 칼럼

### Extra 칼럼

- Distinct
- impossible HAVING
- impossible WHERE
- Not exists
- Using filesort
- Using index(커버링 인덱스)
- Using index for group-by
- Using join buffer 
- Using temporary
- Using where
- Using where with pushed condition

### EXPLAIN EXTENDED (Filtered 칼럼)

### EXPLAIN PARTITIONS (Partitions 칼럼)

- - - -

## MySQL의 주요 처리 방식

### 풀 테이블 스캔

### ORDER BY 처리 (Using filesort)

### GROUP BY 처리

### DISTINCT 처리

### 임시 테이블 (Using temporary)




