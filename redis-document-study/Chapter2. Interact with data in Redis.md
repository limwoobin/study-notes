## __Chapter2. Interact with data in Redis__

### Search and query

Redis Stack 은 향상된 Redis 환경을 다음과 같은 기능들을 제공함

- 풍부한 쿼리 언어
- JSON 및 해시에 대한 증분 인덱싱
- Vector Search
- Full Text Search
- Geospatial queries
- Aggregations

Redis Stack 의 검색 및 쿼리 기능을 사용하면 다음과 같은 기능을 사용할 수 있음
- Document Database
- Vector Database
- Secondary index (보조 인덱스)
- Search engine


#### 기본 구성
- Documents
    - 데이터의 기본 단위
    - 색인 및 검색이 가능한 hash 또는 json 데이터
    - 각 documents 는 key name 으로 식별 가능
- Fileds
    - documents 는 여러개의 field 로 구성됨
    - 문자열, 숫자, geospatial, Vector 와 같은 다양한 유형의 데이터 타입 지정 가능
    - 이 필드를 indexing 하면 해당 값을 기반으로 효율적인 쿼리 및 검색이 가능
- indexing fields
    - 모든 필드를 인덱싱하면 불필요한 오버헤드 발생
    - 인덱싱해야 하는 필드를 유연하게 선택해야함
- Schema
    - 필드가 저장되고 인덱싱되는 방식을 정의

Redis Stack 필드의 유형
- Number Fields
    - 숫자를 저장, 정수 및 부동 소수점 값을 보유할 수 있음, 정렬 가능
    - > FT.CREATE ... SCHEMA ... {field_name} NUMBER [SORTABLE] [NOINDEX]
- Geo Fields
    - 지리 필드 저장, 경도 및 위도와 같은 좌표를 저장하는데 사용
    - FT.CREATE ... SCHEMA ... {field_name} GEO [SORTABLE] [NOINDEX]
- Vector Fields
    - 외부 머신러닝 학습 모델에 의해 생성되는 부동 소수점
    - 텍스트, 이미지 또는 기타 복잡한 기능과 같은 구조화되지 않은 데이터를 나타냄
    - > FT.CREATE ... SCHEMA ... {field_name} VECTOR {algorithm} {count} [{attribute_name} {attribute_value} ...]
    - {algorithm} 은 지정되어야 하며 지원되는 벡터 유사성 색인 알고리즘이어야 함, 지원되는 알고리즘은 다음과 같음
        - FLAT: 무차별 대입 알고리즘.
        - HNSW: 계층적, 탐색 가능한 작은 세계 알고리즘.
- Tag Fields
    - 데이터 태그 또는 레이블 모음을 나타내는 텍스트 데이터를 저장하는 데 사용
    - 카디널리티가 낮다는 특징이 있음
    - 데이터를 구성하고 분류하는 데 유용하므로 특정 태그를 기반으로 문서를 더 쉽게 필터링하고 검색할 수 있음
    - > FT.CREATE ... SCHEMA ... {field_name} TAG [SEPARATOR {sep}] [CASESENSITIVE]
- Text Fields
    - 텍스트 필드를 인덱싱할 때 Redis Stack은 검색 기능을 최적화하기 위해 여러 가지 변환을 수행함
    - 텍스트가 소문자로 변환되어 대소문자를 구분하지 않고 검색할 수 있음
    - 데이터는 토큰화 됨
    - 데이터가 개별 단어나 토큰으로 분할되어 효율적인 전체 텍스트 검색 기능이 가능해짐
    - 특정 필드에 다양한 중요도 수준을 할당하기 위해 텍스트 필드에 가중치를 부여할 수 있음
    - > FT.CREATE ... SCHEMA ... {field_name} TEXT [WEIGHT] [NOSTEM] [PHONETIC {matcher}] [SORTABLE] [NOINDEX] [WITHSUFFIXTRIE]
        - WEIGHT: 가중치, 이는 검색 작업 중 특정 필드에 대해 중요도 수준을 할당 (rdb 의 다중 컬럼 인덱스 느낌이려나 ?)
        - NOSTEM: 형태소 분석이 되지 않음을 나타냄. 토큰화 하고 싶지 않은 데이터를 저장하는데에 유용함
        - PHONETIC {matcher}: 음성 일치 (발음을 기반으로 인덱스를 검색 할 수 있음)
            - e.g. : Jon, John 과 같은 값도 음성으로 검색
        - SORTABLE: 필드를 정렬할 수 있음
        - NOINDEX: 필드가 인덱싱되지 않았음을 의미, 

#### 스키마 정의
인덱스 구조는 스키마에 의해 정의됨

```
FT.CREATE idx 
    ON HASH 
    PREFIX 1 blog:post: 
SCHEMA 
    title TEXT WEIGHT 5.0
    content TEXT
    author TAG
    created_date NUMERIC SORTABLE
    views NUMERIC
```

- 키 이름이 blog:post: 로 시작하는 모든 해시 문서를 색인하는 idx라는 인덱스에 대한 스키마가 정의됨
- 스키마에는 title, content, author, created_date 및 views 필드가 포함됨
- title 및 content 필드가 텍스트 기반, TAG 유형은 작성자 필드에 사용되며, NUMERIC 유형은 created_date 및 views 필드에 사용됨
- titme 필드에는 5.0의 가중치가 할당되어 검색 결과에서 관련성이 높아지며, created_date 는 이 필드를 기준으로 정렬할 수 있도록 SORTABLE 로 표시됨


#### Query
- Selection: 특정 기준을 충족하는 모든 documents를 반환할 수 있음
- Projection: 결과 집합의 특정 필드를 반환하는 데 사용됨, 계산된 필드 값에 매핑할 수도 있음
- Aggregation: 집계는 여러 필드에서 데이터를 수집함

| 유형  | SQL  | Redis Stack   |
| --- | ----- | ------ |
| Selection   | SELECT * FROM bicycles WHERE price >= 1000 | FT.SEARCH idx:bicycle "@price:[1000 +inf]"
   |
| Projection   | SELECT id, price FROM bicycles  | FT.SEARCH idx:bicycle "*" RETURN 2 __key, price
 |
| Projection   | SELECT id, price-price*0.1 AS discounted FROM bicycles  | FT.AGGREGATE idx:bicycle "*" LOAD 2 __key price APPLY "@price-@price*0.1" AS discounted
 |
| Aggregation   | SELECT condition, AVG(price) AS avg_price FROM bicycles GROUP BY condition     | FT.AGGREGATE idx:bicycle "*" GROUPBY 1 @condition REDUCE AVG 1 @price AS avg_price
   |



### Redis programmability

Lua 및 Redis 함수로 Redis 확장하기

Redis 는 서버 자체에서 사용자 정의 스크립트를 수행할 수 있는 인터페이스를 제공, Redis 7 이상에서는 Redis Function,  
Redis 6.2 이하에서는  Lua scripting with the EVAL command 를 사용해 서버를 프로그래밍함

Redis programmability 라는 용어는 서버에서 임의의 사용자 정의 로직을 실행할 수 있는 기능을 의미(스크립트).  
스크립트를 데이터가 존재하는 곳(서버)에서 처리할 수 있게 해준다. 또한 스크립트를 Redis 에 내장하면 네트워크 트래픽을 줄이고 성능을 개선하는데에 도움이 된다.  

eval: 주로 스크립트 언어에서 사용되며, 문자열로 표현된 코드를 실행하는 기능을 제공.
Lua Script: Lua 프로그래밍 언어로 작성된 스크립트로 빠르고 가벼운 특징이 있고 임베디드 시스템이나 Redis 와 같은 데이터베이스에서 스크립트 언어로 사용됨.  
Redis에서 Lua Script 는 `EVAL` or `EVALSHA` 명령어로 사용됨

Spring에서 사용되는 라이브러리중 Redisson 과 Lettuce 의 Redis Lock 사용시 큰 차이중 하나가 바로 이 Lua Script 사용여부이다.
Redisson 에서는 Lua Script 를 사용해 보다 빠른 연산이 된다는 장점이 있다. 
e.g.1 프로젝트에 "EVAL", "EVALSHA" 로 검색시 해당 키워드를 사용하는것을 찾을 수 있음  
e.g.2 RedissonLock.java 의 tryLockInnerAsync method(198 line) 확인

#### Lua Script 로 Redis 명령 호출하는 방법
- redis.pcall(): 함수 호출로 인해 발생한 오류를 스크립트의 실행 컨텍스트로 반환
- redis.call(): 함수 호출로 인해 발생한 오류를 실행한 클라이언트에게 직접 반환

e.g
```
> EVAL "return redis.call('SET', KEYS[1], ARGV[1])" 1 foo bar
OK
```
위 스크립트는 하나의 키 이름과 하나의 값을 입력 인수로 허용. 실행되면 스크립트는 문자열 값 "bar"를 사용하여 SET입력 키 foo 를 설정하는 명령을 호출



## Transaction

Redis 트랜잭션을 사용하면 단일 단계로 명령을 실행할 수 있음, 명령어는 `MULTI, EXEC, DISCARD, WARTCH` 로 사용할 수 있음.  
트랜잭션은 다음과 같이 두가지 중요한 사항을 보장함

- 트랜잭션의 모든 명령은 직렬화되어 순차적으로 실행됨
    - 다른 클라이언트가 보낸 요청은 절대 Redis 트랜잭션이 실행되는 도중에 제공되지 않음
- `EXEC` 명령은 트랜잭션의 모든 명령을 트리거하고 있음
    - 클라이언트가 EXEC 명령을 호출하기 전에 서버와의 연결이 끊어지면 어떠한 작업도 수행되니 않음
    - 대신 `EXEC` 명령이 호출되면 모든 작업이 수행됨


#### 예시

```
> MULTI
OK
> INCR foo
QUEUED
> INCR bar
QUEUED
> EXEC
1) (integer) 1
2) (integer) 1
```
- `MULTI` 명령을 사용해 Redis 트랜잭션을 입력, 이 명령은 항상 OK 로 응답, 이 시점에 사용자가 입력한 명령들은 Queue 에 대기함
- `EXEC` 가 호출되면 모든 명령이 실행됨
- 대신 `DISCARD` 가 호출되면 트랜잭션의 큐가 플러시되고 트랜잭션이 종료됨

#### 트랜잭션 내부의 에러

트랜잭션에서 두 가지 종류의 에러가 발생할 수 있음
- 명령어를 큐에 넣지 못하여 `EXEC` 가 호출되기 전에 오류가 발생할 수 있음
    - e.g) 명령어 구문이 잘못되엇거나, 인수가 잘못되었거나, 명령이 서버의 메모리 제한을 갖도록 구성된 경우
- 잘못된 값의 키에 대해 연산이 수행되는 경우 EXEC 가 호출된 이후 명령이 실패할 수 있음
    - e.g) 문자열에 대해 연산을 호출하는 경우

```
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
MULTI
+OK
SET a abc
+QUEUED
LPOP a
+QUEUED
EXEC
*2
+OK
-WRONGTYPE Operation against a key holding the wrong kind of value
```
다음과 같이 key a 에 문자열 abc 를 넣고 리스트 요소를 제거하는 LPOP 명령어를 사용한 경우엔 잘못된 명령어를 사용했다고 볼 수 있음.  
명령이 실패하더라도 대기열의 다른 모든 명령은 처리된다는 점을 유의해야함

- 잘못된 명령어의 사용에 대해서는 `DISCARD` 명령어가 실행되면서 `Queue` 에 있던 데이터가 지워짐
- 다만, 명령어는 올바르지만 잘못된 자료구조를 사용한 경우에는 에러가 발생하더라도 모든 커맨드를 실행함, 롤백도 되지 않음


#### 트랜잭션의 롤백

롤백은 Redis의 단순성과 성능에 큰 영향을 미치기에 롤백은 지원하지 않음

#### 대기열 삭제

DISCARD는 트랜잭션을 중단하는 데 사용할 수 있음. 이 경우 어떠한 명령도 실행되지 않고 연결 상태가 정상으로 복원됨.

```
> SET foo 1
OK
> MULTI
OK
> INCR foo
QUEUED
> DISCARD
OK
> GET foo
"1"
```

#### Optimistic locking using check-and-set (확인 및 삭제를 위한 낙관적 락)

WATCH된 키는 해당 키에 대한 변경을 감지하기 위해 모니터링 되고 있음.  
EXEC 명령이 실행되기 전에 하나 이상의 watched 키가 수정되면 전체 트랜잭션이 중단되고 EXEC는 트랜잭션이 실패했음을 알리기 위해 Null 응답을 반환함.

```
WATCH mykey
val = GET mykey
val = val + 1
MULTI
SET mykey $val
EXEC
```

위의 코드를 사용하면 경합 조건이 발생하고 WATCH 호출과 EXEC 호출 사이에 다른 클라이언트가 val의 결과를 수정하면 트랜잭션이 실패하여 Null 응답을 반환함.

__더 알아야 할 점__
- `WATCH` 명령어 실행시 타 클라이언트에서 접근한것을 어떻게 감지하여 트랜잭션을 실패처리할까?


## Redis Pub/Sub

- SUBSCRIBE, UNSUBSCRIBE and PUBLISH 는 게시/구독 메시징 패러다임을 구현함
- 대신 발행된 메시지는 어떤 구독자가 있는지 알 수 없는 상태로 채널에 전송됨
- 구독자 또한 어떤 퍼블리셔가 있는지 모르고 채널에만 관심사를 가짐

> SUBSCRIBE channel11 ch:00
`channel11` 및 `ch:00` 채널을 구독하기 위해선 위의 명령어를 실행함

+ 하나 이상의 채널을 구독하는 클라이언트는 다른 채널에 대해  SUBSCRIBE and UNSUBSCRIBE 명령은 할 수 있지만 다른 커맨드를 실행해서는 안됨

#### Redis Pub/Sub 특징
- 메시지를 발행할때 subscriber 측에서 메시지를 정확히 수신했는지 확인하지 않음
- 채널을 구독하는 subscriber 가 없어도 메시지를 채널에 전송하고 날려버림
- kafka 와 달리 메시지를 발행하고 subscriber 가 받았는지 확인하는 작업, 메시지 발행 후 다른 브로커에 메시지를 replication 하는 작업이 없기에 비교적 빠름
    - toss 에서는 이 장점을 이용하여 토스증권의 시세데이터를 노출하는데에 사용하였음 (kafka 에 비해 데이터가 유실될 가능성은 있지만 그만큼 레이턴시가 낮음). 
    https://www.youtube.com/watch?v=SF7eqlL0mjw&list=PL1DJtS1Hv1PiGXmgruP1_gM2TSvQiOsFL&index=14

#### Pub/Sub 명령어

SUBSCRIBE channel [channel ...]
- 지정한 채널로 보내진 메시지를 받습니다. 
- 채널을 여러 개 지정할 수 있습니다.

PSUBSCRIBE pattern [pattern ...]
- 채널을 패턴으로 등록한다. 패턴을 여러 개 등록할 수 있다.
- 패턴은 아래와 같은 glob-style을 지원한다.
    - '?'는 한 글자를 대치한다.   h?llo는 hello, hallo, hxllo 같은 것을 의미한다.
    - '*'은 공백이나 여러 글자를 대치한다.   h*llo는 hllo, heeeello 같은 것을 의미한다.
    - h[ae]llo는 'a' 나 'e'만 올 수 있다.   그래서 hello, hallo는 되고, hillo는 안된다.

```
6379> psubscribe ch*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "ch*"
3) (integer) 1
```

UNSUBSCRIBE [channel [channel ...]]
- SUBSCRIBE로 등록한 채널을 삭제해서, 더 이상 메시지를 받지 않습니다.
- 채널명을 입력하지 않으면 해당 클라이언트에 등록된 모든 채널을 삭제합니다.

PUNSUBSCRIBE [pattern [pattern ...]]
- PSUBSCRIBE로 등록한 패턴을 삭제해서, 더 이상 메시지를 받지 않습니다.
- 패턴명을 입력하지 않으면 해당 클라이언트에 등록된 모든 패턴을 삭제합니다.

PUBLISH channel message
- 메시지를 지정한 채널로 보냅니다.
- 메시지를 받는 클라이언트 수를 리턴합니다.

PUBSUB subcommand [argument [argument ...]]
- PUBSUB 명령은 서버에 등록된 채널이나 패턴을 조회한다.
- 세 가지 subcommand가 있다.   channels, numsub, numpat입니다.
- channels, numsub은 채널 관련 명령이고, numpat은 패턴 관련 명령이다.

PUBSUB channels [pattern]
- Pattern을 입력하지 않으면 해당 서버에 등록된 모든 채널명을 보여준다.

PUBSUB numsub [channel-1 ... channel-n]
- 해당 채널에 등록된 클라이언트 개수를 보여준다.

PUBSUB numpat
- 서버에 등록된 pattern의 개수를 보여준다.

Redis Pub/Sub 의 자료구조 참고. 
http://redisgate.kr/redis/command/pubsub_intro.php