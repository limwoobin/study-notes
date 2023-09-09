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

#### JobExecution
1. 개념
- JobInstance에 대한 한번의 시도를 의미하는 객체 (Job 실행 중 발생한 정보들을 저장하는 객체)
(시작시간, 종료시간, 상태의 속성을 가짐)
- JobInstance와의 관계
  - JobExecution 은 FAILED, COMPLETED 등의 Job 실행 결과를 가짐
  - JobExecution 의 실행 상태 결과가 COMPLETED면 JobInstance의 실행이 완료된 것으로 간주 (재실행 불가)
  - JobExecution 의 실행 상태 결과가 FAILED 면 JobInstance 실행이 완료되지 않은것으로 간주 (재실행 가능)
  (JobParameter 가 동일한 값으로 Job이 실행되더라도 JobIsntance를 계속 실행할 수 있음)

2. 테이블 매핑
- JobInstance 와 JobExecution 은 1:N 의 관계로서 JobInstance의 실행/결과를 가지고 있음

#### Step
1. 개념
- Batch Job 을 구성하는 독립적인 하나의 단계, 실제 배치 처리를 정의하고 컨트롤하는 모든 정보를 가진 도메인 객체
- 단일 테스크 뿐 아닌, 입력 처리 출력 등을 포함하는 설정을 담고 있음
- 배치작업을 어떻게 구성하고 실행할지의 Job 세부 작업을 Task 기반으로 설정/명세함
- 모든 Job은 하나 이상의 Step으로 구성

2. 기본 구현체
- TaskletStep: 가장 기본이 되는 클래스, Tasklet 타입의 구현체를 제어
- PartionStep: 멀티 스레드 방식으로 Step 을 여러 개로 분리해서 실행
- JobStep: Step 내에서 Job 을 실행함
- FlowStep: Step 내에서 Flow 를 실행

[Step]
- Step 을 실행시키는 execute 메소드
- 실행 결과 상태는 StepExecution 에 저장됨

[AbstractStep]
- name: Step 이름
- startLimit: Step 실행 제한 횟수
- allowStartIfComplete: Step 실행이 완료된 후 재 실행 여부
- stepExecutionListener: Step 이벤트 리스너
- jobRepository: Step 메타데이터 저장소

#### StepExecution
기본 개념
- Step 에 대한 한번의 시도를 의미하는 객체, Step 실행 중 발생한 정보들을 저장
(시작시간, 종료시간, 상태, commit count, rollback count ...)
- Step 이 매번 시도될 때마다 생성, 각 Step 별로 생성
- Job 이 재시작 하더라도 이미 성공적으로 완료된 Step은 재실행되지 않고, 실패한 Step 만 실행됨
- 이전 단계 Step이 실패하여 현재 Step을 실행하지 않았다면 StepExcecution을 생성하지 않음, Step이 실제로 실행되었을때만 StepExecution을 생성함
- JobExecution 과의 관계
  - Step의 StepExecution 이 모두 정상적으로 완료되어야 JobExecution이 정상적으로 완료됨
  - Step의 StepExecution 중 하나라도 실패하면 JobExecution은 실패함

BATCH_STEPEXECUTION 테이블과 매핑
- JobExecution 와 StepExecution 은 1:N 관계
- 하나의 Job에 여러 개의 Step으로 구성했을 경우 각 StepExecution 은 하나의 JobExecution을 부모로 가짐

