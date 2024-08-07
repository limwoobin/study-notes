# Virtual Thread

JVM 자체적으로 내부 스케줄링을 통해서 사용할 수 있는 경량의 스레드를 제공한다. 
하나의 Java 프로세스가 수십만~ 수백만개의 스레드를 동시에 실행할 수 있게끔 설계되었다.

**KLT: 커널 레벨 스레드**
**ULT: 유저 레벨 스레드**

## 기존 Java Thread Model
- Native Thread 로 생성시 JNI 를 통해 커널 영역을 호출해(System Call) KLT를 생성하고 
ULT 와 매핑하여 작업을 수행
- Thread 는 기본적으로 최대 2MB 의 스택 메모리 사이즈를 가지기에 컨텍스트 스위칭 시 메모리 이동량이 큼, 시스템 콜을 이용하기에 생성비용도 적지 않음
- KLT와 ULT가 1:1 로 매핑

## Virtual Thread
- 플랫폼 스레드와 가상 스레드로 나뉨
- 플랫폼 스레드 위에서 가상 스레드가 번갈아 가며 실행되는 형태로 동작
- 컨텍스트 스위칭 비용이 저렴함
ㄴ JVM 에 의해 생성되기에 시스템 콜과 같은 커널 영역의 호출이 적음
ㄴ 메모리 크기도 일반 스레드의 1%에 불과함

### Virtual Thread 의 구조

- 플랫폼 스레드의 기본 스케줄러는 ForkJoinPool 을 사용함
	ㄴ 이 스케줄러는 platform thread pool 을 관리하고, 
	virtual thread 의 작업 분배 역할을 함
- KLT(1) : ULT(1) : Virtual Thread(N) 의 구조로 사용
	ㄴ KLT와 Virtual Thread 사이의 ULT를 platform thread 라고 함

+ 플랫폼 스레드는 OS 를 Wrapping 한것으로 자바의 전통적인 스레드 모델
ㄴ 즉, 이 전통적인 스레드 모델 위에서 가상 스레드가 동작하는거임

virtual thread 가 가지고 있는 객체들
- carrierThread: 실제 작업을 수행시키는 platform thread, workQueue 를 가지고 있음
- scheduler: ForkJoinPool 을 가지고있음, carrier thread 의 Pool 역할을 하고 가상 스레드의 스케줄링을 담당
- runContinuation: virtual thread 의 실제 작업 내용(Runnable) 을 가지고 있음

### Virtual Thread 의 동작 원리

1. 실행될 virtual thread 의 작업인 runContinuation 을 carrier thread 의 workQueue 에 push 함
2. workQueue 에 있는 runContinuation 은 ForkJoinPool(scheduler) 에 의해
work stealing 방식으로 carrier thread 에 의해 처리됨
3. 처리되던 runContinuation 들은 I/O, Sleep 으로 인한 interrupt 혹은 작업 완료시 
workQueue 에서 pop 되어 park 과정에 의해 다시 힙 메모리로 돌아감



#### 가상스레드는 CPU Bound 작업에 비효율적

#### Thread Local 사용 주의
- 가상 스레드에선 Thread Local 을 사용하는것이 좋지는 않다
- 가상 스레드는 수시로 생성되고 소멸되며 스위칭되기에 엄청 많은 스레드를 운용할수있도록 설계되었다
- 그래서 항상 크기를 작게 유지하는게 좋다 (가볍게 이용하는것이 목적)
ㄴ 이때 Thread Local 의 크기가 커질수록 메모리 이슈가 생길 수 있다
ㄴ 그렇기에, Thread Local 을 사용하더라도 무거운 객체를 저장하지 않고 불변객체나 글로벌 캐시를 사용하는 등으로 우회해야 한다

#### Pinned Issue
- 가상 스레드 내에서 synchronized, parallelStream, 혹은 네이티브 메서드를 사용하면 
가상 스레드가 캐리어 스레드에 고정되는 문제가 있음
- 이런 경우, 가상 스레드의 장점인 Non-blocking 방식으로 동작하지 않음
- synchronized 대신 ReentrantLock을 사용하는 것이 좋음


### Platform Thread vs Virtual Thread

- Metadata size
ㄴ Platform Thread: 약 2kb(OS 별 차이 있음)
ㄴ Virtual Thread: 200 ~ 300B

- Metadata size
ㄴ Platform Thread: 미리 할당된 Stack 사용
ㄴ Virtual Thread: 필요시마다 Heap 사용

- Metadata size
ㄴ Platform Thread: 1~10 us (마이크로 세컨즈, 커널 영역에서 발생)
ㄴ Virtual Thread: 나노세컨즈