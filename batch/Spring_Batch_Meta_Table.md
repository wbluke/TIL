# Spring Batch Meta Table
#TIL/batch

## 참고
[3. Spring Batch 가이드 - 메타테이블엿보기](https://jojoldu.tistory.com/326)

---

### BATCH_JOB_INSTANCE

- Job Parameter에 따라 생성되는 테이블
	- 같은 배치, 다른 Job Parameter는 이력이 남고, 같은 배치, 같은 Job Parameter는 이력이 남지 않는다. (에러 처리)
	- 단, 같은 Job Parameter로 실패한 후 다시 실행시켰을 때에는 에러가 나지 않는다. Spring Batch에서는 동일 Job Parameter로 성공한 기록이 있어야 재수행을 하지 않는다.

### BATCH_JOB_EXECUTION

- JOB_INSTANCE와 부모-자식 관계
	- 자신의 부모 JOB_INSTANCE가 성공하거나 실패했던 모든 이력을 가지고 있다.

### BATCH_JOB_EXECUTION_PARAM

- BATCH_JOB_EXECUTION 테이블 레코드가 생성될 때 받은 Job Parameter를 가지고 있다.

### BATCH_STEP_EXECUTION

- BATCH_JOB_EXECUTION과 비슷하게 Job 내부에서 실행되는 Step 정보를 확인할 수 있다.

### BATCH_JOB_EXECUTION_CONTEXT, BATCH_STEP_EXECUTION_CONTEXT

- 중간에 배치 Job 혹은 Step이 중단되더라도 이어서 실행할 수 있게 해주는 정보를 가지고 있다.
