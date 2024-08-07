# ETC

## 신입이 어떻게 대규모 트래픽 처리 경험을 쌓아요? https://www.youtube.com/watch?v=b4Ro_2cK9V8

채용공고의 대규모 트래픽 경험 우대인 경우에서 대규모 트래픽 경험이 없어도 이 부분을 대비할 수 있는 방법은?

- 대규모의 규모에 대해서는 정량적이지 않기때문에 먼저 내가 생각하는 대규모에 대해서 정의한다
- 대규모에 대해서 어떻게 어느정도를 측정할지? (Traffic, Latency, CPU, Memory, Disk ...)
	- 모니터링 도구를 이용해 파악 가능
	- Jmeter, Ngriner 와 같은 성능테스트를 통해 테스트 해볼 수 있을듯
- 부하를 만들어보고 생기는 병목지점에 대해 SQL 튜닝 혹은 캐싱을 이용해 개선해보는 경험기를 만들어보자
- 인터뷰어는 공공데이터들을 이용해서 전처리 및 시각화를 하는 프로젝트를 진행했다고함



## N + 1
@OneToOne 에서 연관관계 주인이 아닌곳에서 엔티티를 조회하게 되면 
@OneToOne 관계의 엔티티가 LazyLoading 으로 설정되어있더라도 N+1 이 발생하게된다.

이유 > 연관관계 주인이 아닌곳에서는 상대의 FK 를 모르기에 상대객체가 NULL 일 수가 있다.
그래서 이때 객체입장에선 이 객체를 프록시로 할당할지 NULL 로 할당할지 알 수 없는 상황이다.
그래서 Lazy 로 동작하지 않고 바로 Eager 로 쿼리를 실행해 해당 객체에 대해 어떤 값인지 알게끔 처리된다 
+ (프록시는 Null 을 감쌀수 없어서 둘 중 어떤 값으로든 넣기 애매한 상황이 발생한다)



## message

at_least_once : 이벤트가 발생했을 때, 해당 이벤트가 최소 1번은 이벤트가 발행되어 
처리되는 것을 보장을 한다는 의미
- 메시지 유실을 방지하기 위해 중복으로 메시지를 처리하는 것을 허용하는 것

at_most_once : 이벤트가 발생했을 때, 최대 1번만 이벤트 메시지가 발행되게 한다는 의미
- 메세지 유실을 허용하고 중복으로 이벤트 처리를 방지하기 위한 것

## Spring Event

- 이벤트를 왜 쓰나?
의존성을 분리하기 위해
e.g) 유저를 저장할때 유저 히스토리도 저장하는데 이때 히스토리를 저장하는 책임은 분리한다
왜? 히스토리 저장에서 장애가 나게되면 이 히스토리때문에 유저도 저장하지 못하는 이슈가 있다.
즉, 유저 저장과 히스토리 저장의 의존성(결합도)를 분리하는것

- TaskExecutor 를 직접 빈으로 등록하는 이유
- @Transactional
- Spring AOP 와 프록시
- 언제 사용하면 좋을지


## DNS TCP, UDP
- DNS: 사람이 읽을 수 있는 도메인을 머신이 읽을수 있는 IP 주소로 변환
	- e.g) naver.com -> 192.0.2.44
- DNS 는 기본적으로 UDP 를 사용함, 왜냐면 DNS 는 일반적으로 512 바이트 미만의 데이터를 처리하고
신뢰성보다 속도가 중요한 서비스이기에
- 하지만 간혹 TCP 를 사용하는 경우도 있음
	- 메시지 크기가 512 바이트보다 큰 경우 (DNS 는 512byte 크기 제한이 있음)
	- zone 정보 동기화 하는 경우(master 와 slave 간에) -> zone-transfer
	- zone-transfer: DNS 트랜잭션의 한 유형으로 여러대의 DNS 서버간에 DNS 데이터베이스를 복제하는데 사용되는 방법


## Redis 야무지게 사용하기

#### 사용하면 안되는 커맨드
- `keys *` 사용하지 않기, `keys * -> scan` 으로 대체
- `hgetall -> hscan`
- `del -> unlink`

#### 변경하면 장애를 막을 수 있는 기본 설정값

`STOP-WRITES-ON-BGSAVE-ERROR = NO`
- 기본값은 yes
- RDB 파일 저장 실패시 redis 로의 모든 write 를 불가능하게 하는것
- 이 설정을 꺼두는게 오히려 장애를 막을 수 있는 방법일 수 있다

`MAXMEMORY-POLICY = ALLKEYS-LRU`
- 메모리가 가득 찼을때 MAXMEMORY-POLICY 정책에 의해 키가 관리됨
	- 기본값은 noeviction : 삭제안함 이다, 이는 메모리가 가득차면 아무액션을 하지않기에 에러가 발생할듯
	- allkeys-lru 같은 설정을 쓰자
	- volatile-lru 는 ttl 이 있는 key 만 삭제하기에 ttl 이 없다면 동일상황이 발생할 수 있음
- redis 를 캐시로 사용할때 Expire Time 설정 권장

#### Persistence / 복제 사용 시 MaxMemory 설정 주의
- Copy-on-Write 로 인해 메모리 복재할때 메모리를 두개 가량 사용하는 경우가 발생할 수 있음
- 그렇기에 Persistence / 복제 사용시 MAXMEMORY 는 실제 메모리의 절반으로 설정하는게 안전
e.g) 4GB -> 2G (2048MB)


#### Memory 관리

물리적으로 사용되고 있는 메모리를 모니터링
used_memory 값 보다 used_memory_rss 값을 보는게 더 중요하다

- used_memory: 논리적으로 Redis 가 사용하는 메모리
- used_memory_rss: OS 가 Redis 에 할당하기 위해 사용한 물리적 메모리 양
- 삭제되는 키가 많으면 fragmentation 증가
	- 특정 시점에 피크를 찍고 다시 삭제되는 경우
	- TTL 로 인한 eviction 이 많이 발생하는 경우

#### 레디스 클러스터에서는 레디스 트랜잭션을 사용할 수 없는듯??

#### 레디스 동작 및 스레드
- 레디스는 싱글스레드이다, 정확히는 메인 스레드1개와 별도 스레드 3개로 동작한다.
- 클라이언트 커맨드를 처리하는 부분은 이벤트 루프를 이용한 싱글 스레드로 동작한다.
	- 멀티스레드 어플리케이션에서 요구되는 동기화나 잠금 메커니즘이 없어 빠르게 요청을 처리할 수 있다.
	- 별도 스레드는 일반적으로 I/O 작업이나 데이터 백업작업을 진행함

<hr />
<br />

## 리눅스 중요 명령어

### IP 를 확인하는 명령어, 자신의 Public IP 는 어떻게 확인하나요?
- 대표적으로는  curl ifconfig.co, curl ifconfig.me

### CURL 의 옵션별 사용법

### Domain 의 IP 를 조회하는 명령어는?
- nslookup

### 웹서버 혹은 DB 같은 서버들을 확인하는 방법
- telnet 과 ping 을 구분하는 의도

### 내 서버의 서버가 잘 떠있는지, 현재 DB 커넥션 등을 확인하는 명령어는?
- netstat 명령어와 그 옵션들을 잘 아는지에 대한 의도임
- netstat -lntp
- netstat -an | grep "port"

### 리눅스에서 특정 프로세스를 확인하는 명령어는?
- ps 명령어
- ps -ef | grep ""
- ps -aux | grep ""

### 리눅스에서 CPU, Memory, Disk 등 시스템 정보를 확인하는 명령어는?
- top, sar, free, df ...

### 리눅스에서 서비스들은 어떻게 관리되고, 그와 연관된 명령어는?
- service, sysctl, systemd 명령어

### 리눅스 파일권한 체계를 이해하고 있는지?
- chmod, chown 에 대한 질문

<hr />
<br />

# Kafka 

Transactional Outbox Pattern은 분산 시스템에서 데이터베이스 트랜잭션과 메시지 큐를 조합하여 데이터 일관성과 메시지 전송의 원자성을 보장하는 패턴

- 일반적으로 Mysql 과 이벤트를 하나의 트랜잭션으로 처리할거같은데 debezium 이라는 
라이브러리를 사용한다

<hr />
<br />

# CPU Bound , I/O Bound

Burst
- 어떤 현상이 짧은 시간안에 집중적으로 일어나는 일
- 계속되는 작업

CPU Burst
- 프로세스가 CPU 에서 한번에 연속적으로 실행되는 시간
- 메모리에 올라가있는 프로세스가 자신의 차례가 되어 CPU 에서 실행될때 연속적으로 실행되는 시간

I/O Burst
- 프로세스가 I/O 작업을 요청하고 결과를 기다리는 시간

CPU Bound 프로세스
- CPU Burst 가 많은 프로세스 -> CPU 를 많이 사용하는 프로세스
- e.g) 동영상 편집 프로그램(그래픽 작업), 머신러닝 프로그램, 엄청난 계산이 필요한 어플리케이션
- 성능 향상을 위해서는 scale-up 이 주로 사용됨

I/O Bound 프로세스
- I.O Burst 가 많은 프로세스 -> I/O 를 많이 사용하는 프로세스
- 입출력 시스템(키보드, 마우스 등등), 디스크 드라이브, 네트워크 작업, 주변 장치와 같은 리소스에 의존하는 경우가 I/O Bound
- 입출력 시스템에서 데이터를 읽고 쓰고 정보를 기다리는 모든 애플리케이션은 I/O 바운드로 간주됨
ㄴ 워드 프로세싱 시스템, 웹 어플리케이션, 파일 복사 및 다운로드 등
- e.g) 백엔드 API 서버 -> DB 서버, 캐시서버 등등 요청을 보내는 경우가 많음
- 성능 향상을 위해서는 스레드 수를 최적화한다(주로 scale-out)

+ 프로세스를 실행하면 CPU Burst 와 I/O Burst 가 번갈아가면서 나타난다
+ 일부 프로그램이나 작업은 CPU 와 I/O 에 모두 속한다

적절한 스레드의 개수는?
CPU Bound
- 스레드가 너무 많으면 스레드끼리 코어를 사용하기 위해 경합하면서 컨텍스트 스위칭이 발생해 오버헤드가 커질 수 있음
- 그러니 코어의 개수 혹은 코어의 개수에서 크게 벗어나지 않는 선(1개 정도?)에서 스레드 개수를 설정하는것이 효유렂ㄱ

I/O Bound
- 정해진 가이드는 없어서 경험적으로 최적의 스레드 개수를 찾아야 한다
- 중요한것은, 스레드가 무조건 많다고 해서 성능이 좋은것은 아니다, 오히려 컨텍스트 스위칭이라는 오버헤드가 발생해서 성능이 저하될 수 있다

- https://www.baeldung.com/cs/cpu-io-bound

<br />
<hr />

# park() vs unpark()
- 다른말로 마운트, 언마운트라고 한다

park()
- 스레드를 멈추게한다(블록시킨다)
- 스레드가 깨어날때까지 멈추어있는데
- 스레드가 park 상태에 있을때는 다른 스레드에 의해 unpark 되거나 인터럽트되기 전까지는 깨어나지 않는다.

unpark()
- 스레드를 깨운다
- 특정 스레드를 깨워서 다시 실행가능상태로 만든다
- unpark 는 스레드가 park 메서드에 의해 멈춰있지 않은 경우에도 호출할 수 있음
이러한 경우에는 쓰레드가 park 메서드가 호출될때까지 멈추지않고 계속 실행될 수 있게 해줌

# Thread 의 상태

- NEW
- WAITING
- TIME_WAITING
- RUNNABLE
- BLOCKED
- TERMINATED

스레드가 interrupt 되는 경우
- 스레드가 I/O 작업을 수행하거나, 네트워크 작업, 파일 읽기/쓰기 작업을 수행하는 동안 인터럽트가 발생할 수 있음, 이때 InterruptedException 이 발생할 수 있음
- 아직 스레드가 작업중인 상황에서 ExecutorService 가 shutdown 되는 경우
- 작업 시간 초과 처리
	- 특정 작업이 제한된 시간 내에 완료되지 않으면 해당 작업을 중단시키고 다른 처리를 해야 할 때.
	- 예를 들어, 네트워크 요청이 일정 시간 내에 응답하지 않으면 이를 중단시키고 타임아웃 처리를 하는 경우.

## InterruptedException

어플리케이션에서 사용하는 경우
- Future.get(), BlockingQueue.get() 과 같은 블로킹 작업을 할때 타임아웃이 걸려 
작업을 중단시킬때

시스템상 스레드를 사용하며 InterruptedException 가 발생할 수 있는 경우

- 시스템에서 스레드를 종료해야 하는 경우 스레드가 인터럽트 될 수 있음
(예를들면, 어플리케이션이 종료되거나, 시스템이 강제로 스레드를 종료해야 하는 경우)
- 시스템이 메모리 부족이나 다른 자원 제한으로 인해 정상적으로 스레드를 실행할 수 없는 경우
스레드를 인터럽트 할 수 있음
- JVM 이 종료되거나 재시작되는 경우 JVM 내의 모든 스레드가 인터럽트 될 수 있음

# ForkJoinPool (with parallelStream())

기본적으로 parallelStream 은 ForkJoinPool 의 commonPool 을 사용함
이는 하나의 commonPool 을 다른 parallelStream 들과 공유하게 되고
해당 스레드 풀을 사용하는 다른 스레드들과 서로 영향을 줄수있다
-> 이를 해결하기 위해서는 각 parallelStream 마다 커스텀 ForkJoinPool 을 사용하여 스레드풀을 분리하면 해결할 수 있다

별도 ForkJoinPool 스레드풀 생성시 정석은 CPU 코어수를 기준으로 생성하는것이다.
`ForkJoinPool customForkJoinPool = new ForkJoinPool(Runtime.getRuntime().availableProcessors());
`

기본으로 사용하는 CommonForkJoinPool 은 정적이기에 메모리 누수가 발생하지 않지만
커스텀하게 생성한 ForkJoinPool 은 참조해제되지않거나 GC에 수집되지 않을 수 있다
그렇기에 사용 후 명시적으로 종료해줘야 한다 -> `forkJoinPool.shutdown();`

parallelStream 은 분할되는 작업의 단위가 균등하게 나누어져야 하며, 나누어지는 작업의 비용이 높지않아야 순차적방식보다 효율적으로 이뤄질 수 있음
array, ArrayList 같이 전체 사이즈를 알 수 있는 경우엔 분할처리가 빠르고 비용이 적지만,
LinkedList 같이 사이즈를 정확히 알수없는 데이터구조는 분할되지 않고 순차처리를 하므로 성능효과를 보기 어렵다

병렬로 처리되는 작업이 독립적이지 않다면 수행성능에 영향이 있을 수 있음
예로, stream 중간연산으로 sorted(), distinct() 같은 작업이 있는 경우
내부적으로 상태에 대한 변수를 각 작업들이 공유해야 한다
이런경우엔 순차처리가 더 효율적이다

<br />
<hr />

# CAP 정리

Consistency(데이터 일관성)
- 일관성을 가진다는 것은 모든 데이터를 요청할때 응답으로 가장 최신의 변경된 데이터를 리턴 혹은 실패를 리턴
- 모든 읽기에 대해 DB 노드가 항상 동일한 데이터를 가지고 있어야함

Availability(가용성)
- 모든 요청에 대해 정상적인 응답을 함
- 클러스터 노드의 일부에서 장애가 발생해도 READ 나 WRITE 등의 동작은 항상 성공적으로 리턴되야함

Partition Tolerance(분할 내성)
- 시스템 일부가 망가져도 시스템이 계속 동작해야함
- 분할 내성이란 노드간 통신장애가 발생해도 동작해야함
- DB의 파티션과는 다른 개념

CP(Consistence & Partition Tolerance)
- 어떤 상황에서도 안정적으로 시스템은 운영되지만 Consistence 가 보장되지 않는다면 에러를 반환
- 매순간 Read/Write 에 따른 정합성이 일치해야 함

AP(Availability & Partition Tolerance)
- 어떤 상황에서도 안정적으로 시스템은 동작함
- 데이터와 상관없이 안정적인 응답을 받을 수 있음
- 데이터의 정합성에 대한 보장은 불가능

# Tomcat 의 ThreadPool

server.tomcat.threads.max
- Thread Pool에서 사용할 최대 스레드 개수, 즉 동시에 처리할 수 있는 최대 요청 수
- 실제 Active User 수 이다
- 기본값은 200

server.tomcat.threads.min-spare
- Thread Pool에서 최소한으로 유지할 Thread 개수
- Java ThreadPool 의 coreSizePool 개념과 유사한듯
- 기본값은 10

server.tomcat.max-connections
- 동시에 처리할 수 있는 최대 Connection 의 개수, 즉 톰캣 서버가 동시에 허용하는 TCP/IP 연결 수
- 기본값은 8192
- 사실상 서버의 실질적인 동시 요청처리개수라고 생각할 수 있다.
- 연결 개수가 max-connections 에 도달했더라도, 운영체제는 accept-count 만큼 추가로 연결을 수락한다

server.tomcat.accept-count
- max-connections 이상의 요청이 들어왔을 때 사용하는 요청 대기열 Queue 의 사이즈 
- 기본값은 100
- 부적절한 요청들을 필터링하는 데 필요!
- accept-count 가 10이라면, 이후 추가요청인 11개 부터는 요청을 거절하거나 timeout 에러가 발생함

<hr />
<br />

# JVM 

자바코드 실행과정
1. Java Source(.java) -> Java Compiler 를 통해 Java ByteCode 로 변환(.class)
2. 컴파일된 Java ByteCode 를 JVM 의 클래스로더에게 전달
3. 클래스로더는 동적로딩을 통해 필요한 클래스들을 로딩 및 링크하여 Runtime Data Area 로 전달 (즉, JVM 의 메모리에 올림)
4. Execution Engine(실행엔진)은 JVM 메모리에 올라온 바이트 코드들을 명령어 단위로 하나씩 가져와서 실행

Execution Engine (실행엔진)
- 메모리에 적재된 바이트코드(.class) 를 기계어로 변환해 명령어 단위로 읽어 실행하는 역할을 함
(.class 파일의 코드를 JVM 이 읽을 수 있게 실행)
- 인터프리터와 JIT(Just In Time) 컴파일러를 같이 사용함
- 인터프리터는 바이트코드를 한줄씩 읽어서 실행하는 방식
- JIT 는 바이트코드의 전체 또는 일부를 네이티브로 컴파일하고 직접 실행하는 방식
- 프로그램 실행 초기에는 인터프리터 방식으로 빠르게 실행하고 실행중엔 JIT 컴파일러가 분석을 통해 자주 사용되거나 성능이 중요한 부분을 식별해 네이티브 코드로 컴파일함

클래스 로더(Class Loader)
- JVM 내로 클래스 파일(*.class)을 동적으로 로드하고, 링크를 통해 배치하는 작업을 수행하는 모듈
- 즉 로드된 바이트 코드를 엮어 JVM 메모리 영역인 Runtime Data Area 에 배치함


-- Runtime Data Area 영역 --

Method(Static) Area
- JVM이 동작하고 클래스가 로딩될 때 생성되며 JVM이 읽어들인 클래스 변수(Static 변수), 생성자(constructor)와 메소드(method) 등을 저장하는 공간
- 모든 스레드에 공유됨
- Method 영역의 데이터는 스레드가 종료될때까지 메모리에 남아있음
- Method 영역의 데이터는 프로그램이 종료될때까지 어디서든 접근/사용 가능

Method Area 내의 Constant Pool 영역(상수 풀)
- 메서드 영역에 존재하는 별도의 관리 영역
- 각 클래스/인터페이스 마다 별도의 constant pool 테이블이 존재하는데, 클래스를 생성할때 참조해야할 정보들을 상수로 가지고 있는 영역(레퍼런스 저장)
- 즉 클래스, 인터페이스, 상수, 메서드 등에 대한 레퍼런스 저장

Heap Area
- 참조형(Reference Type) 데이터 타입을 갖는 객체(인스턴스), 배열 등과 같은 런타임 데이터를 저장하기 위한 영역이다.
- Stack 영역과 다르게 단 하나의 heap 영역만 생성되어 어느곳에서나 접근 가능하다. (공유됨)
- 단, Heap 영역에 있는 객체들을 가리키는 참조를 위한 주소값은 Stack 에 포함됨
- Heap 영역은 Stack 과 다르게 보관된 데이터가 호출이 끝나더라도 삭제되지 않고 유지됨
- 어떤 참조변수도 Heap 영역의 인스턴스를 참조하지 않는다면, GC 에 의해 메모리에서 제거됨

Stack Area
- 메소드 내에서 정의하는 기본 자료형(Primitive Type - int, double, byte, long, boolean 등)에 해당되는 지역변수, 매개변수의 데이터 값, Heap 영역의 객체 참조값이 저장되는 공간이다.
- 각 스레드마다 독립적인 Stack 영역을 갖는다
- 임시적으로 사용되는 변수나 정보들이 저장되는 영역
- 메소드가 실행될때 Stack 영역 안에 스택 프레임이 생기고 Stack 영역내의 메소드를 호출
- 메소드가 호출될때 메모리에 할당되고 메소드가 메모리에서 사라짐

PC Register
- 스레드마다 별도로 존재하며 스레드가 시작될때 생성됨
- 현재 실행중인 JVM 명령의 주소값을 가짐

Native Method Stack
- 자바가 아닌 언어로 작성된 네이티브 코드를 위한 메모리 영역
- 즉, 기계어로 작성된 프로그램을 실행시키는 영역

<br />

## Heap

Young Generation: 생명 주기가 짧은 객체를 GC 대상으로 하는 영역 (Minor GC 가 발생)
- Eden: 객체가 생성되면 최초로 저장되는 곳, 정기적인 쓰레기 수집 후 살아남은 객체들은 Survuvor 로 이동
- Survivor0 / Survivor1: 각 영역이 채워지게 되면, 살아남은 객체는 비워진 Survivor 로 순차적으로 이동

Old Generation: 생명주기가 긴 객체를 GC 대상으로 하는 영역, Young Generation 에서 마지막까지 살아남은 객체가 이동 (Major GC 가 발생)



<hr />
<br />

# 우아콘 Kafka Streams 를 활용한 이벤트 스트림 처리 삽질기

- 스트림 프로세서는 결국 카프카 컨슈머임
- 컨슈머는 멀티스레드로 병렬처리가 가능함
- 리밸런싱에 대한 전략을 고민하자.
	- EAGER: 모든 컨슈머의 파티션 할당을 끊고 재조정
	- COOPERATIVE: 다운타임을 최소화하는 대신, EAGER 보다 여러번/오래 수행함
- 리밸런싱 시간/횟수를 줄이기 위해 검토해볼만한 설정값
	- 리밸런싱 전략 (eager / cooperative)
	- 컨슈머 처리속도/처리량 조절을 위한 레코드 폴링 주기/양 (max.poll.interval.ms / max.poll.records)
	- 컨슈머 그룹에서 탈락하지 않기 위한 heartbeat 기준 (session.timeout.ms / heartbeat.interval.ms)
	- 초반 리밸런싱 대기시간 (group.initial.rebalance.delay.ms)
	- 파티션/컨슈머 멤버수가 과하게 많지는 않은가
	- 근본적으로는 이벤트 처리 로직 최적화 ?
- 상태저장소 내부 토픽의 Retention, Delete Policy 도 잘 관리하자. 
얘네때문에 카프카 디스크 부하가 발생할 수 있다
	- cleanup.policy

모니터링 

- 컨슈머 랙 모니터링에 대한것은 Burrow 를 이용함
- JMX ??

<hr />
<br />

# 우아콘 대규모 트랜잭션을 처리하는 배민 주문시스템

조인 연산으로 인한 성능 저하
- mongodb 를 이용한 역정규화를 통해 이슈 해결
- RDB 와 mongodb 에 모두 저장, 주문시 동기화함

대규모 트랜잭션에서 쓰기연산의 한계치
- 샤드 클러스터를 도입하려했지만 AWS 에선 샤딩을 지원하지 않음
- 애플리케이션 샤딩을 하기로 함, key based 샤딩 전략을 사용함

애그리거트는 어떻게 할지?
- mongodb 를 통해 조회를하고있어 해당 디비를 이용해 편하게 사용함

<hr />
<br />

# 모니터 락

구성요소

- mutex
	- mutex lock 을 획득하지 못한 스레드는 큐에 들어간 후 대기상태로 전환
	- mutex lock 을 쥔 스레드가 lock 을 반환하면 락을 기다리며 큐에 대기상태로 있던 스레드 중 하나가 락을 획득하여 실행
	- mutex lock 을 대기하는 스레드는 entry queue 에서 대기

- condition variable
	- waiting queue 를 가짐
	- 조건이 충족되길 기다리는 스레드들이 대기상태로 기다리는곳
	- signal: waiting queue 에서 기다리는 스레드 중 하나를 깨움
	- broadcast: waiting queue 에서 기다리는 모든 스레드들을 깨움

- entry queue : critical section 에 진입을 기다리는 큐
- waiting queue : 조건이 충족되길 기다리는 큐


<br />
<hr />
