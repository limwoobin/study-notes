# **JVM(Java Virtual Machine) 정리**

## **JVM 구조**

![jvm-image-1](https://user-images.githubusercontent.com/28802545/159151398-e9320096-3a54-45eb-83e4-04f03579a3b8.png)

## **구성 요소**

### **Class Loader**

- .java 파일들은 자바 컴파일러(JAVAC) 에 의해 .class 확장자를 가지는 자바 바이트 코드로 변환됨
- 클래스 로더는 생성된 클래스 파일들을 JVM 메모리 영역인 Runtime Data Area 로 적재함

### **Excution Engine(실행 엔진)**

- 메모리에 적재된 .class 파일(바이트 코드)들을 기계어(Binary Code)로 변경해 명령어 단위로 수행
- 바이트 코드를 운영체제에 맞게 해석해주는 역할을 수행(Byte Code -> Binary Code)
  - Interpreter(인터프리터)  
    명령어를 한줄씩 해석하고 실행
  - JIT(Just In Time)  
    Byte Code를 native code 로 변경 후 실행

### **Runtime Data Area(실행 데이터 영역)**

JVM 이 운영체제로부터 할당받은 메모리 영역, JVM 은 이 영역에 어플리케이션에서 사용하는 데이터들을 적재

- ### **메소드 영역(Method Area or Static Area)**

  - 클래스 멤버변수의 이름, 데이터 타입, 접근 제어자 정보같은 필드 정보 저장
  - 메소드 이름, 리턴 타입, 파라미터같은 메소드 정보 저장
  - 인터페이스 and 클래스 정보, 상수, static 변수 정보 저장
  - 모든 스레드 공유 가능

- ### **힙 영역(Heap Area)**

  - 런타임시 동적을 할당하여 사용하는 객체(new Instance) 및 배열 저장
  - 메소드 영역에 로드된 클래스만 생성 가능
  - Garbage Collector가 관리하는 메모리 영역
  - 모든 스레드 공유 가능

- ### **스택 영역(Stack Area)**

  - 메소드 호출시 생성되는 스레드 수행정보를 Frame을 통해 저장
  - 메소드가 호출되면 메소드와 메소드 정보는 Stack에 쌓이게 되며, 메소드 호출이 종료될떄 Stack point에서 제거됨
  - 메소드 정보는 해당 메소드의 매개변수, 지역변수, 임시변수 그리고 어드레스 등을 저장하고 메소드 종료시 메모리 공간이 사라짐
  - 각 스레드 별로 소유(다른 스레드에선 접근 X)

- ### **PC Register**

  - 현재 실행중인 JVM 주소 정보를 저장
  - 수행해야할 CPU 명령어 위치 정보를 저장
  - 각 스레드 별로 소유

- ### **Native Method Stack**

  - Java 외의 언어로 작성된 네이티브 코드를 위한 메모리 ex) JNI(Java Native Interface)
  - C/C++ 등의 코드를 수행하기 위한 스택
  - 각 스레드 별로 소유

![jvm-image-2](https://user-images.githubusercontent.com/28802545/159151937-d9a55c24-39c2-4be0-a047-be5bd6510341.png)

<br><br>

Heap Area의 세분화된 메모리 구조

- Eden - 최초에 new 키워드를 통해 객체가 생성되는 영역, Young Generation, Minor GC 수행
- Survivor1, Survivor2 - Eden 영역에서 GC 대상이 아닌 객체들을 전달받아 저장하는 영역, Young Generation, Minor GC 수행
- Old - Young Generation 영역에서 GC대상이 아니어서 살아남은 객체들이 이동하는 영역, Major GC 수행 영역
- Permanent - 클래스 로더에 의해 적재된 클래스 저장(jdk1.8 부터는 Metaspace)

  - Metaspace 는 Native memory 영역으로 기존 Perm 은 JVM 으로부터 크기가 강제되었지만, Metaspace는 OS가 자동으로 크기를 조정함

  ![jvm-image-3](https://user-images.githubusercontent.com/28802545/159152845-305ac03a-6317-4e49-a420-c8fd3062ce57.jpg)

<br>

### **Garbage Collector**

Heap Area에 생성된 객체들 중 참조되지 않은 객체들을 탐색 후 제거하는 역할  
가비지 컬렉터가 동작시 JVM 어플리케이션이 실행을 멈추게 되는데 이를 `STOP-THE-WORLD` 라고 한다.  
이는 시스템 동작이 멈추는 작업이기에 비용이 크다, GC 튜닝을 한다는 이야기는 즉 `STOP-THE-WORLD` 시간을 단축한다는 이야기이다.

**Garbage Collection 종류**

- Major Garbage Collection - Old 영역에서 발생하는 GC
- Minor Garbage Collection - Young 메모리 영역에서 발생하는 GC
- Full Garbage Collection - 메모리 전체를 수행하는 GC

**Garbage Collector 종류**

- Serial Garbage Collector
- Parallel Garbage Collector
- CMS Garbage Collector
- G1 Garbage Collector(JDK7)
- Epsilon Garbage Collector(JDK11)
- Z garbage collector(JDK11)
- Shenandoah Garbage Collector(JDK12)

<br>

<hr>

## **Jvm heap size option**

### **xms**

Java 힙의 초기 크기를 제어합니다.  
이 값을 적절하게 조절하면 가비지 콜렉션의 오버헤드를 줄여서 서버 응답 시간 및 처리량을 개선합니다.  
일부 응용프로그램의 경우, 이 옵션에 대한 기본 설정이 너무 낮아서 사소한 가비지 콜렉션의 수가 높아질 수 있습니다.

- 기본값: 50MB, 이 기본값은 31bit, 64bit 구성 모두에 적용됨
- 사용법: -Xms256m 은 초기 힙 크기를 256MB 로 설정

<br>

### **xmx**

Java 힙의 최대 크기를 제어.  
이 변수를 늘리면 Application Server에서 사용 가능한 메모리가 늘어나고 가비지 컬렉션 빈도가 줄어듬.  
이 설정을 늘리면 서버 응답 시간 및 처리량이 개선 될 수 있음  
설정을 사용 가능한 시스템 메모리 이상으로 늘리면 시스템 페이징 및 상당한 성능 감소를 유발할 수 있음.
그러나 이 설정을 늘리면 가비지 콜렉션이 발생할 때 해당 콜렉션의 지속 기간이 늘어남. 이 설정은 Application Server 인스턴스에 대해 사용 가능한 시스템 메모리 이상으로 증가해서는 안됨.

- 기본값: 256MB, 이 기본값은 31bit, 64bit 구성 모두에 적용됨
- 사용법: -Xmx512m 은 최대 힙 크기를 512MB로 설정

> Sun HotSpt JVM 계열에서는 최초 크기와 최대 크기를 동일하게 부여할 것을 권장한다. 크기의 동적인 변경에 의한 오버 헤들를 최소화하기 위해서이다.
