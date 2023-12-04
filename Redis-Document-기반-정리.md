# Redis Study with Documentation

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


## __Chapter2. Interact with data in Redis__

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





