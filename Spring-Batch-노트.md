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


#### Job 실행 순서 및 설정
1. @Configuration 선언 - 하나의 배치 job을 정의하고 빈 설정
2. JobBuilderFactory - job 을 생성하는 빌더 팩토리
3. StepBuilderFactory - Step 을 생성하는 빌더 팩토리
4. Job - bean name으로 job 생성
5. Step - bean name으로 Step 생성
6. Tasklet - Step 안에서 단일 테스크로 수행되는 로직 구현
7. Job 구동 -> Step 을 실행 -> Tasklet 을 실행


#### DB 스키마 생성
- batch_step_execution_context
- batch_job_instance
- batch_step_execution
- batch_job_execution
- batch_job_execution_params
- batch_job_execution_context
