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

#### StepContribution
기본 개념
- 청크 프로세스의 변경 사항을 버퍼링 한 후 StepExecution 상태를 업데이트하는 도메인 객체
- 청크 커밋 직전에 StepExecution 의 apply 메소드를 호출하여 상태를 업데이트 함
- ExitStatus 의 기본 종료코드 외 사용자 정의 종료코드를 생성해 적용할 수 있음

#### ExecutionContext
- 프레임워크에서 유지/관리하는 키/값으로 된 컬렉션, StepExecution 또는 JobExecution 객체의 상태를 저장하는 공유 객체
- DB에 직렬화 된 값으로 저징됨 ["key": "value"]
- 공유 범위
  - Step 범위: 각 Step의 StepExecution에 저장되며 Step간 서로 공유 안됨
  - Job 범위: 각 Job의 JobExecution에 저장되며 Job간 서로 공유 안됨, 해당 Job의 Step간에 서로 공유됨
- Job 재시작시 이미 처리한 row 데이터는 건너뛰고 이후 수행하도록 할 때 상태 정보로 활용됨

- ExecutionContext 구조
`Map<String, Object> map = new ConcurrentHashMap`

#### JobRepository
기본 개념
- 배치 작업 중의 정보를 저장하는 저장소 역할
- Job이 언제 수행되었고, 언제 끝났으며, 몇 번이 실행되었고 실행에 대한 결과 등의 배치 작업의 수행과 관련된. 
모든 meta data를 저장함 (jobLauncher, Job, Step 구현체 내부에서 CRUD 기능을 처리)

#### JobLauncher
기본 개념
- 배치 Job을 실행시키는 역할
- Job, Job Parameters 를 인자로 받고 요청된 배치 작업을 수행한 후 최종 client에게 JobExecution을 반환
- 스프링 부트 배치가 구동되면 JobLauncher 빈이 자동 생성됨
- Job 실행: JobLauncher.run(Job, JobParameters)

동기적 실행: taskExecutor 를 SyncTaskExecutor 로 설정할 경우 (기본값은 SyncTaskExecutor)
비도기적 실행: taskExecutor를 SimpleASyncTaskExecutor 로 설정할 경우

<hr>

#### JobBuilderFactory

JobBuilderFactory
- JobBuilder 를 생성하는 팩토리 클래스, get method 제공
- jobBuilderFactory.get("jobName")

JobBuilder
- job 을 구성하는 설정 조건에 따라 두 개의 하위 빌더 클래스를 생성하고 실제 Job 생성을 위임
- SimpleJobBuilder
  - SimpleJob 을 생성하는 Builder Class
  - Job 실행과 관련된 여러 설정 API 를 제공
- FlowJobBuilder
  - FlowJob 을 생성하는 Builder 클래스
  - 내부적으로 FolowBuilder 를 반환함으로 Flow 실행과 관련된 여러 설정 API 를 제공

#### SimpleJob
- SimpleJob 은 Step 을 실행시키는 Job 구현체, SimpleJobBuilder 에 의해 생성됨
- 여러 단계의 Step 으로 구성할 수 있으며, Step 을 순차적으로 실행 시킴
- 모든 Step 의 실행이 성공적으로 완료되어야 Job 이 성공적으로 완료됨
- 맨 마지막에 실행한 Step 의 BatchStatus 가 Job 의 최종 BatchStatus 가 된다

```
return jobBuilderFactory.get("jobName")     // JobBuilder 를 생성하는 팩토리, Job 의 이름을 매개변수로 받음
  .start(Step)                              // 처음 실행 할 Step 설정, 최초 한번 설정, 이 메서드를 실행하면 SimpleJobBuilder 반환
  .next(Step)                               // 다음에 실행할 Step 설정, 횟수는 제한이 없으며 모든 next() 의 Step 이 종료되면 Job 이 종료됨
  .incrementer(JobParametersI=incrementer)  // JobParameter 의 값을 자동 증가해주는 설정
  .preventRestart(true)                     // Job 의 재 시작 가능 여부 설정, 기본값은 true
  .validator(JobParameterValidator)         // JobParameter 를 실행하기 전에 올바른 구성이 되었는지 검증하는 JobParametersValidator 설정
  .listener(jobExecutionListener)           // Job 라이프 사이클의 특정 시점에 콜백 제공받도록 JobExecutionListener 설정
  .build();                                 // SimpleJob 생성
```

#### SimnpleJob - start/next


#### SimpleJob - validator
- Job 실행에 필요한 파라미터를 검증하는 용도
- DefaultJobParametersValidator 구현체를 지원, 좀 더 복잡한 제약조건이 있다면 인터페이스를 직접 구현할 수 있음
- DefaultJobParametersValidator 는 기본 생성자, required, optional 생성자 2개를 배열로 받음

#### SimpleJob - preventRestart
- Job의 재 시작 여부를 결정
- 기본 값은 true, false 는 이 Job 을 재시작 하지 않는다는 의미
- Job 이 실패해도 재시작이 안되며 Job을 재시작하려고 하면 JobRestartException 이 발생
- 재 시작과 관련 있는 기능으로, Job을 처음 실행하는 것과는 아무 관계가 없음
- Job 의 실행이 처음이 아닌 경우, Job 의 성공/실패와는 관계없이 오직 preventRestart 설정 값에 따라 실행 여부를 판단함

#### Incrementer
- JobParameters 에서 필요한 값을 증가시켜 다음에 사용될 JobParameters 오브젝트를 리턴
- 기존의 JobParameter 변경없이 Job 을 여러 번 시작하고자 할때
- RunIdIncrementer 구현체를 지원하며 인터페이스를 직접 구현할 수 있음

<br />

#### StepBuilderFactory
- StepBuilder 를 생성하는 팩토리 클래스, get(String name) 메서드 제공
- StepBuilderFactory.get("stepName") > "stepName" 으로 Step 을 생성

#### StepBuilder
- Step 을 구성하는 설정 조건에 따라 다섯 개의 하위 빌더 클래스를 생성하고 실제 Step 생성을 위임
- TaskletStepBuilder : TaskletStep 을 생성하는 기본 빌더 클래스
- SimpleStepBuilder : TaskletStep 을 생성하여 내부적으로 청크기반의 작업을 처리하는 ChunkOrientedTasklet 클래스를 생성
- PartionStepBuilder : PartionStep 을 생성하여 멀티 스레드 방식으로 Job 을 실행
- JobStepBuilder : JobStep 을 생성하여 Step 안에서 Job 을 실행
- FlowStepBuilder : FlowStep 을 생성하여 Step 안에서 Flow 를 실행

<br>

#### TaskletStep
- 기본 개념
  - 스프랭 배치에서 제공하는 Step 의 구현체, tasklet 을 실행시키는 도메인 객체
  - RepeatTemplate 를 사용해 Tasklet 구문을 트랜잭션 경계 내에서 반복해서 실행함
  - Task 기반과 Chunk 기반으로 나누어 Tasklet 을 실행함

- Task vs Chunk 기반 비교
  - 스프링 배치에서 Step 의 실행 단위는 크게 2가지로 나누어짐
  - chunk 기반
    - 하나의 큰 덩어리를 n개씩 나나ㅜ어 실행한다는 의미, 대량 처리를 하는 경우 효과적으로 설계 됨
    - ItemReader, ItemProcessor, ItemWriter 를 사용해 청크 기반 전용 Tasklet 인 ChunkOrientedTasklet 구현체가 제공됨
  - Task 기반
    - 청크 기반보다 단일 작업 기반으로 처리되는 것이 더 효율적인 경우
    - 주로 Tasklet 구현체를 만들어 사용
    - 대량 처리를 하는 경우 chunk 기반에 비해 더 복잡한 구현이 필요함

```java
public Step batchStep() {
  return stepBuilderFactory.get("batchStep")  // StepBuilder 를 생성하는 팩토리, Step 이름을 매개변수로 받음
    .tasklet(Tasklet)                         // Tasklet 클래스 설정
    .startLimit(10)                           // Step 의 실행 횟수 설정, 설정한 만큼 실행되고 초과시 오류 발생, 기본값은 Integer.MAX_VALUE
    .allowStartIfComplete(true)               // Step 의 성공, 실패와 상관없이 항상 Step을 실행하기 위한 설정
    .listener(StepExecutionListener)          // Step 의 라이프사이클의 특정 시점에 콜백 제공받도록 StepExecutionListener 설정
    .build();                                 // TaskletStep 을 생성
}

```

#### tasklet()
- Tasklet 타입의 클래스를 설정한다
  - Tasklet
    - Step 내에서 구성되고 실행되는 도메인 객체, 주로 단일 테스크를 수행하기 위한 것
    - TaskletStep 에 의해 반복적으로 수행, 반환값에 따라 계속 수행 또는 종료함
    - RepeatStatus - Tasklet의 반복 여부 상태 값
      - RepeatStatus.FINISH: Tasklet 종료, RepeatStatus를 null 로 반환하면 RepeatStatus.FINISHED 로 해석됨
      - RepeatStatus.CONTINUABLE: Tasklet 반복
      - RepeatStatus.FINISH 가 리턴되거나 실패가 던져지기 전까지는 TaskletStep 내에서 while 문에 의해 반복적으로 호출됨 (무한루프 주의)
- 익명 클래스 혹은 구현 클래스를 만들어 사용
- 이 메소드를 실행하게 되면 TaskletStepBuilder 가 반환되어 관련 API를 설정할 수 있음
- Step 에 오직 하나의 Tasklet 설정이 가능, 두개 이상을 설정한 경우에는 마지막에 설정한 객체가 실행됨

#### startLimit()
- Step의 실행 횟수를 조정할 수 있음
- Step 마다 설정할 수 있음
- 설정 값을 초과해서 다시 실행하려 하면 StartLimitExceededException 이 발생
- start-limit 의 디폴트는 Integer.MAX_VALUE

#### allowStartIfComplete()
- 재시작 가능ㅎ란 Job 에서 Step의 이전 성공 여부와 상관없이 항상 step을 실행하기 위한 설정
- 실행마다 유효성을 검증하는 Step이나 사전작업이 꼭 필요한 Step 등에서 사용될 수 있음
- 기본적으로 COMPLETED 상태를 가진 Step은 Job 재시작시 실행하지 않고 스킵한다
- allow-start-if-complete 가 true 로 설정된 step은 항상 실행함

#### JobStep
- Job 에 속하는 Step 중 외부의 Job 을 포함하고 있는 Step
- 외부의 Job 이 실패하면 해당 Step 이 실패하므로 결국 최종 기본 Job 도 실패한다
- 모든 메타데이터는 기본 Job 과 외부 Job 별로 각각 저장된다
- 커다란 시스템을 작은 모듈로 쪼개고 Job의 흐름을 관리하고자 할 때 사용할 수 있음

```java
public Step jobStep() {
  return stepBuilderFactory.get("jobStep")        // StepBuilder 를 생성하는 팩토리, Step 의 이름을 매개변수로 받음
    .job(job)                                     // JobStep 내에서 실행 될 Job 설정, JobStepBuilder 반환
    .launcher(jobLauncher)                        // Job 을 실행할 JobLauncher 설정
    .parametersExtractor(jobParametersExtractor)  // Step 의 ExecutionContext 를 Job 이 실행되는데 필요한 JobParameters 로 변환
    .build();                                     // JobStep 을 생성
}
```

<br>

#### FlowJob
- Step 을 순차적으로만 구성하는 것이 아닌 특정한 상태에 따라 흐름을 전환하도록 구성할 수 있으며 FlowJobBuilder 에 의해 생성됨
  - Step 이 실패하더라도 Job 은 실패로 끝나지 않도록 해야 하는 경우
  - Step 이 성공 했을때 다음에 실행해야 할 Step 을 구분해서 실행해야 하는 경우
  - 특정 Step 은 전혀 실행되지 않게 구성해야 하는 경우
- Flow 와 Job 의 흐름을 구성하는데만 관여하고 실제 비즈니스 로직은 Step 에서 이루어짐
- 내부적으로 SimpleFlow 객체를 포함하고 있으며 Job 실행 시 호출함

```java
public Job batchJob() {
  return jobBuilderFactory.get("batchJob")
    .start(Step)                                  // Flow 시작하는 Step 설정
    .on(String pattern)                           // Step 의 실행 결과로 돌려받는 종료상태(ExitStatus)를 캐치하여 매칭하는 패턴, TransitionBuilder 반환
    .to(Step)                                     // 다음으로 이동할 Step 지정
    .stop() / fail() / end() / stopAndRestart()   // Flow 를 중지 / 실패/ 종료하도록 Flow 종료
    .from(Step)                                   // 이전 단계에서 정의한 Step 의 Flow 를 추가적으로 정의
    .next(Step)                                   // 다음으로 이동할 Step 지정
    .end()                                        // build() 앞에 위치하면 FlowBuilder를 종료하고 SimpleFlow 객체 생성
    .build();
}
```

### Transition

#### BatchStatus
- JobExecution 과 StepExecution 의 속성, Job 과 Step 의 종료 후 최종 결과 상태가 무엇인지 정의
- SimpleJob
  - 마지막 Step 의 BatchStatus 값을 Job 의 최종 BatchStatus 값으로 반영
  - Step 이 실패할 경우 해당 Step 이 마지막 Step 이 된다
- FlowJob
  - Flow 내 Step 의 ExitStatus 값을 FlowExecutionStatus 값으로 저장
  - 마지막 Flow 의 FlowExecutionStatus 값을 Job 의 최종 BatchStatus 값으로 반영

- `ABANDONED` : 처리를 완료했지만 성공하지 못한 단계와 재시작시 건너 뛰어야 하는 단계

#### ExitStatus
- JobExecution 과 StepExecution 의 속성으로 Job 과 Step 의 실행 후 어떤 상태로 종료되었는지 정의
- 기본적으로 ExitStatus 는 BatchStatus 와 동일한 값으로 설정됨

- SimpleJob
  - 마지막 Step 의 ExitStatus 값을 Job 의 최종 ExitStatus 값으로 반영
- FlowJob
  - Flow 내 Step 의 ExitStatus 값을 FlowExecutionStatus 값으로 저장
  - 마지막 Flow 의 FlowExecutionStatus 값을 Job 의 최종 ExitStatus 값으로 반영

#### FlowExecutionStatus
- FlowExecution 의 속성으로 Flow 의 실행 후 최종 결과 상태가 무엇인지 정의
- Flow 내 Step 이 실행되고 나서 ExitStatus 값을 FlowExecutionStatus 값으로 저장
- FlowJob 의 배치 결과 상태에 관여함

#### Transition
- Flow 내 Step 의 조건부 전환(전이)을 정의함
- Job 의 API 설정에서 on(String pattern) 메소드를 호출하면 TransitionBuilder 가 반환되어 Transition Flow 를 구성할 수 있음
- Step 의 종료상태(ExitStatus) 가 어떤 pattern 과도 매칭되지 않으면 스프링 배치에서 예외를 발생하고 Job 은 실패
- transition 은 구체적인 것부터 그렇지 않은 순서로 적용됨

#### API
- on(String pattern)
  - Step 의 실행 결과로 돌려받는 종료상태(ExitStatus) 와 매칭하는 패턴 스키마, BatchStatus 와 매칭하는 것이 아님
  - pattern 과 ExitStatus 와 매칭이 되면 다음으로 실행할 Step 을 지정할 수 있음
  - 특수문자는 두 가지만 허용
    - '*' : 0개 이상의 문자와 매칭, 모든 ExitStatus 와 매칭됨
    - '?' : 정확히 1개의 문자와 매칭
    - ex) 'c*t' 는 'cat', 'count' 모두 매칭가능, 'c?t' 는 'cat' 에는 매칭되지만 'count' 에는 매칭되지 않음
- to()
  - 다음으로 실행할 단계를 지정
- from()
  - 이전 단계에서 정의한 Transition 을 새롭게 추가 정의함

#### Job 을 중단하거나 종료하는 Transition API
- Flow 가 실행되면 FlowExecutionStatus 에 상태값이 저장되고 최종적으로 Job 의 BatchStatus 와 ExitStatus 에 반영됨
- Step 의 BatchStatus 및 ExitStatus 에는 아무런 영향을 주지 않고 Job 의 상태만을 변경함

- stop()
  - FlowExecutionStatus 가 STOPPED 상태로 종료되는 transition
  - job 의 BatchStatus 와 ExitStatus 가 STOPPED 로 종료됨
- fail()
  - FlowExecutionStatus 가 FAILED 상태로 종료되는 transition
  - Job 의 BatchStatus 와 ExitStatus 가 FAILED 로 종료됨
- end()
  - FlowExecutionStatus 가 COMPLETED 상태로 종료되는 transition
  - Job 의 BtachStatus 와 ExitStatus 가 COMPLETED 로 종료됨
  - Step 의 ExitStatus 가 FAILED 이더라도 Job 의 BatchStatus 가 COMPLETED 로 종료되도록 가능하며 이 때 Job 의 재시작은 불가능함
- stopAndRestart(Step or Flow or JobExecutionDecider)
  - stop() transition 과 기본 흐름은 동일
  - 특정 step 에서 작업을 중단하도록 설정하면 중단 이전의 step 만 COMPLETED 저장되고 이후의 step 은 실행되지 않고 STOPPED 상태로 job 종료
  - Job 이 다시 실행되었을때 실행해야 할 step 을 restart 인자로 넘기면 이전에 COMPLETED 로 저장된 step 은 건너뛰고 중단 이후 step 부터 다시 시작


#### 사용자 정의 ExitStatus
- ExitStatus 에 존재하지 않는 exitCode 를 새롭게 정의해서 설정
- StepExecutionListener 의 afterStep() 메서드에 Custom exitCode 생성 후 새로운 ExitStatus 반환
- Step 실행 후 완료 시점에서 현재 exitCode 를 사용자 정의 exitCode 로 수정할 수 있음


#### JobExecutionDecider
- ExitStatus 를 조작하거나 StepExecutionListener 를 등록할 필요 없이 Transition 처리를 위한 전용 클래스
- Step 과 Transition 역할을 명확히 분리해서 설정할 수 있음
- Step 의 ExitStatus 가 아닌 JobExecutionDecider 의 FlowExecutionStatus 상태값을 새롭게 설정해서 반환함


#### SimpleFlow
- 스프링 배치에서 제공하는 Flow 의 구현체, 각 요소(Step, Flow, JobExecutionDecider) 들을 담고 있는 State 를 실행시키는 도메인 객체
- FlowBuilder 를 사용해서 생성, Transition 과 조합하여 여러 개의 Flow 및 중첩 Flow 를 만들어 Job 을 구성할 수 있음

Flow
- getName(): Flow 이름 조회
- Stage getState(String stateName): state 명으로 State 타입 반환
- FlowExecution start(FlowExecutor executor): Flow 를 실행시키는 start 메소드, FlowExecutor 를 넘겨주어 실행을 위임함,
실행 후 FlowExecution 을 반환
- FlowExecution resume(String stateName, FlowExeucutor executor): 다음에 실행할 State 를 구해서 FlowExecutor 에게 실행을 위임함
- Collection<State> getStates(): Flow 가 가지고 있는 모든 State 를 Collection 타입으로 반환

SimpleFlow
- String name: Flow 이름
- String startState: State들 중에 처음으로 실행할 State
- Map<String, Set<StateTransaction>> transitionMap: State 명으로 매핑되어있는 Set<StateTransition>
- Map<String, State> stateMap: State 명으로 매핑되어 있는 State 객체
- List<StateTransition> stateTransitions: State 와 Transition 정보를 가지고 있는 StateTransition 리스트


#### FlowStep
- Step 내에 Flow 를 할당하여 실행시키는 도메인 객체
- flowStep 의 BatchStatus 와 ExitStatus 는 Flow 의 최종 상태값에 따라 결정됨

#### @JobScope / @StepScope

Scope
- 스프링 컨테이너에서 빈이 관리되는 범위
- singleton, prototype, request, session 등이 있으며 기본은 singleton 으로 생성됨

- @JobScope, @StepScope
  - Job 과 Step 의 빈 생성과 실행에 관여하는 스코프
  - 프록시 모드를 기본으로 함 `@Scope(value = "job", proxyMode = ScopeProxyMode.TARGET_CLASS)` 
  - 해당 스코프가 선언되면 빈이 생성이 어플리케이션 실행시점이 아닌 빈의 실행시점에 이루어짐
    - @Values 를 주입해서 빈의 실행시점에 값을 참조할수 있음 (Lazy Binding)
    - @Values 를 사용할 경우 빈 선언문에 @JobScope, @StepScope 를 정의하지 않으면 오류를 발생하므로 반드시 선언해야 함
  - 병렬처리 시 각 스레드마다 생성된 스코프 빈이 할당되기에 thread safe 하게 실행이 가능함

@JobScope
- Step 선언문에 정의한다
- @Value: jobParameter, jobExecutionContext 만 사용 가능

@StepScope
- Tasklet 이나 ItemReader, ItemWriter, ItemProcessor 선언문에 정의함
- @Value: jobParameter, jobExecutionCOntext, stepExecutionContext 사용가능

#### @JobScope / @StepScope 아키텍처



<br /><br />
<hr>

- ItemReader 는 읽어올 데이터가 없다면 명시적으로 null 을 반환해야함  
그래야 Reader 가 종료된것을 알 수 있음, 그렇지 않으면 Reader 가 무한루프가 될 수도 있음


## Batch 주의사항

Spring Boot, Mysql 환경에서 CursorItemReader 사용시 주의사항
- JPACursorItemReader 는 모든 데이터를 메모리에 들고오기에 위험할 수 있음
- Jdbc, Hibernate Cursor 는 적은 메모리로 많은 용량을 처리할 수 있음,  
하지만 `mysql-connector-java` 드라이버 사용시 그냥은 cursor fetch 기능을 사용못함.  
그래서 `jdbc:mysql://localhost/?useCursorFetch=true` 와 같이 `useCursurFetch` 옵션을 넣어주어야만 Mysql 의 cursor 기능을 사용할 수 있음
- 위 설정이 누락되면 일반 쿼리처럼 모든 데이터를 한번에 가져오게 됨


