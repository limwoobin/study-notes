# Spring Batch Note init

## 개요

#### 배치 핵심 패턴
Read - 데이터베이스, 파일, 큐에서 다량의 데이터 조회한다.
Process - 특정 방법으로 데이터를 가공한다.
Write - 데이터를 수정된 양식으로 다시 저장한다

#### 배치 시나리오
- 배치 프로세스를 주기적으로 커밋
- 동시 다발적인 Job 의 배치 처리, 대용량 병렬 처리
- 실패 후 수동 또는 스케줄링에 의한 재시작
- 의존관계가 있는 step 여러 개를 순차적으로 처리
- 조건적 Flow 구성을 통한 체계적이고 유연한 배치 모델 구성
- 반복, 재시도, Skip 처리


#### @EnableBatchProcessing
- 총 4개의 설정 클래스를 실행시킴, 스프링 배치의 모든 초기화 및 실행 구성이 이루어짐
- 빈으로 등록된 모든 Job을 검색해서 초기화와 동시에 Job을 수행하도록 구성

1. BatchAutoConfiguration
- 스프링 배치가 초기화 될 때 자동으로 실행되는 설정 클래스
- Job을 수행하는 JobLauncherApplicationRunner 빈을 생성
2. SimpleBatchConfiguration
- JobBuilderFactory와 StepBuilderFactory 생성
- 스프링 배치의 주요 구성 요소 생성 (프록시 객체로 생성)
3. BatchConfigureConiguration
 a. BasicBatchConfigure
   - SimpeBatchConfiguration 에서 생성한 프록시 객체의 실제 대상 객체를 생성하는 설정 클래스
   - 빈으로 의존성 주입받아 주요 객체들을 참조받아 사용 가능
 b. JpaBatchConfigurer
   - JPA 관련 객체를 생성하는 설정 클래스
 c. 사용자 정의 BatchConfigurer 인터페이스를 구현하여 사용 가능

 [순서] @EnableBatchProcessing -> SimpleBatchConfiguration -> BatchConfigureConiguration (BasicBatchConfigure, JpaBatchConfigurer) -> BatchAutoConfiguration

<hr>


### Job 실행 순서 및 설정
1. @Configuration 선언 - 하나의 배치 job을 정의하고 빈 설정
2. JobBuilderFactory - job 을 생성하는 빌더 팩토리
3. StepBuilderFactory - Step 을 생성하는 빌더 팩토리
4. Job - bean name으로 job 생성
5. Step - bean name으로 Step 생성
6. Tasklet - Step 안에서 단일 테스크로 수행되는 로직 구현
7. Job 구동 -> Step 을 실행 -> Tasklet 을 실행


### DB 스키마

#### [Job 관련 테이블]
batch_job_instance
- job 실행시 jobInstance 정보가 저장, job_name과 job_key를 키로 하여 하나의 데이터가 저장
- 동일한 job_name과 job_key로 중복 저장될 수 없음
batch_job_execution
- job 실행정보가 저장, job의 생성, 시작, 종료 시간, 실행상태, 메시지 등을 관리
batch_job_execution_params
- job과 함께 실행되는 jobParameter 정보 저장
batch_job_execution_context
- job의 실행동안 여러가지 상태정보, 공유 데이터를 직렬화(json)해서 저장
- Step간 서로 공유 가능

#### [Step 관련 테이블]
batch_step_execution
- Step의 실행정보가 저장, 생성, 시작, 종료 시간, 실행상태, 메시지 등을 관리
batch_step_execution_context
- Step의 실행동안 여러가지 상태정보, 공유 데이터를 직렬화(json)해서 저장
- Step 별로 저장, Step간에 공유할 수 없음

<hr>
<br>

### Batch 도메인 

#### Job
- 배치 계층 구조에서 가장 상위의 개념, 하나의 배치작업 자체를 의미
- Job Configuration을 통해 생성되는 객체 단위, 배치작업을 어떻게 구성하고 실행할지 명세해놓은 객체
- 스프링 배치가 기본 구현체를 제공
- 여러 Step을 포함하고 있는 컨테이너, 반드시 한 개 이상의 Step으로 구성해야함

AbstractJob
name - Job 이름
restartable: 재시작 여부(기본값은 true)
JobRepository: 메타데이터 저장소
JobExecutionListener: job 이벤트 리스너
JobParametersIncrementer: jobParameter 증가기
JobParametersValidator: jobParameter 검증기 
SimpleStepHandler: Step 실행 핸들러

SimpleJob
- 순차적으로 Step을 실행시키는 Job
- 모든 Job에서 유용하게 사용할 수 있는 표준 기능

FlowJob
- 특정 조건과 흐름에 따라 Step을 구성하여 실행시키는 Job
- Flow 객체를 실행시켜 작업을 진행

#### JobInstance
- Job의 작업 실행을 나타냄
(예를 들어, 하루에 한 번 씩 배치 Job이 실행된다면 매일 실행되는 Job을 JobInstance로 표현)
- 처음 시작하는 Job, JobParameter일 경우 새로운 JobInstance 생성
- 이전과 동일한 Job, JobParameter일 경우 이미 존재하는 JobInstance 리턴
(내부적으로 JobName + JobKey(jobParameters의 해시값) 을 가지고 JobInstance 객체를 얻음)
- Job과는 1:N 관계
- Job_name, Job_key가 동일한 데이터는 중복해서 저장할 수 없음

#### JobParameter
1. 개념
- Job을 실행할 때 함께 포함되어 사용되는 파라미터를 가진 도메인 객체
- 하나의 Job에 존재할 수 있는 여러개의 JobInstance를 구분하기 위한 용도
- JobParameters 와 JobInstance는 1:1 관계

2. 생성 및 바인딩
- Application 실행 시 주입 
(java -jar LogBatch.jar requestedDate=20231010)
- 코드로 생성
(jobParameterBuilder, DefaultJobParametersConverter)
- SpEL 이용
(@Value("#{jobParameter[requestDate]}")), @JobScope, @StepScope 선언 필수

3. BATCH_JOB_EXECUTION_PARAM 테이블과 매핑
- JOB_EXECUTION 과 1:N 관계

