## __Chapter1. Understand Redis data types__

#### Strings
- Redis 문자열은 텍스트, 직렬화된 객체, 바이너리 배열을 포함한 바이트 시퀀스를 저장함.
- 캐싱에도 사용되고 카운터를 구현하고 비트 연산을 수행할 수 있는 추가 기능도 지원함.
- 이미지도 문자열로 저장 가능

`INCRBY` 명령어를 이용한 카운터도 만들 수 있음  
`INCR, DECR, DECRBY` 와 같은 카운터 명령어는 원자적으로 설계되어 동시에 실행되더라도 경쟁상태로 빠지지 않음  

`read-increment-set` 작업은 다른 모든 클라이언트가 동시에 명령을 실행하지 않는 동안 수행함

성능
- 대부분의 문자열 연산은 O(1) 이다.
- GETRANGE 명령은 O(n) 이 될 수 있으니 주의

<hr>

#### Lists
- Redis Lists 는 다음과 같은 용도로 자주 사용됨
    - Stack, Queue 구현
    - 백그라운드 작업자 시스템용 대기열 관리를 구축

성능
- Head, Tail에서의 작업은 O(1)
- 요소 중간에서의 작업은 일반적으로 O(n)의 복잡도를 가짐

<hr>

#### Sets
중복되지 않는 문자열의 모음(Set 자료구조와 유사)이며 정렬되지 않음

- 고유 항목을 추적가능(예: 특정 블로그 게시물에 액세스하는 모든 고유 IP 주소 추적).
- 관계를 나타낼 수 있음(예: 주어진 역할을 가진 모든 사용자 집합).
- 교집합, 합집합, 차이 등 일반적인 집합 연산을 수행합니다.

성능
- 추가, 제거, 항목이 집합 멤버인지 확인하는 등 대부분의 집합 작업은 O(1
- 대규모 Sets의 경우 명령을 실행할 때 주의, `SMEMBERS` 이 명령은 O(n)이며 단일 응답으로 전체 세트를 반환

<hr>

#### Hashes
필드-값 쌍의 컬렉션으로 구성된 레코드 유형, 해시를 사용하면 무엇보다도 기본 개체를 나타내고 카운터 그룹을 저장할 수 있음.

성능
- 대부분의 Redis 해시 명령은 O(1)
- HKEYS, HVALS, HGETALL 과 같은 몇 가지 명령은 O(n)

<hr>

#### SortedSets
점수에 따라 정렬된 고유 문자열(구성원) 모음, 둘 이상의 문자열에 동일한 점수가 있는 경우 문자열 순으로 정렬 

- 리더보드. 예를 들어, SortedSets 를 사용하면 대규모 온라인 게임에서 가장 높은 점수를 순서대로 나열한 목록을 쉽게 유지할 수 있음.
- 속도 제한기. 특히, 과도한 API 요청을 방지하기 위해 정렬된 세트를 사용하여 속도 제한기를 구축할 수 있음

성능
- 대부분의 연산은 O(log(n)), 여기서 n 은 멤버 수
- ZRANGE 를 사용하여 큰 반환 값(예: 수만 개 이상)을 받을때는 주의해야함 이 명령의 시간 복잡도는 O(log(n) + m). 여기서 m 은 반환된 결과 수.

<hr>

#### Redis Streams
Stream 은 Redis 5.0 에 새로 도입된 데이터 유형, log 처럼 `append-only` 로 동작함

Redis Streams 는 다음과 같은 항목에서 사용될 수 있음(Kafka 와 비슷)
- 이벤트 소싱
- 모니터링
- 알림

Stream 의 특징
- 레디스 스트림에서는 소비자(Consumer)를 지정해서 데이터를 읽을 수 있음
- 그 소비자가 데이터를 제대로 처리했는지 확인하는 방법을 제공
- 만약 제대로 처리하지 못했다면 다른 소비자에게 할당해서 처리하도록 하는 방법을 제공함

성능
- 스트림에 항목을 추가하는 것은 O(1)
- 단일 항목에 액세스하는 것은 O(n)
