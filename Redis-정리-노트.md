# **Redis**

## 자료구조

## lettuce
- setnx메서드를 이용해 사용자가 직접 스핀락형태로 구성됨
스핀락은 redis에 많은 부하를 주어서 응답시간을 느리게 할 수 있음  
추가로 만료시간을 제공하지 않음

## redisson

### RLock
redis에서 제공하는 lock 을 이용할 수 있는 lib  
`tryLock(long waitTime, long leaseTime, TimeUnit unit)` 
- waitTime: 락을 사용할 수 있을때 까지 기다리는 시간
- leaseTime: 락이 만료되는 시간
- TimeUnit: 시간단위

<br>

pub / sub 시스템을 사용함  
락이 해제될때마다 subscribe중인 client들에게 `락 획득을 시도해도 된다`라는 메시지를 보낸다.

## Redis의 백업 방식

### RDB
Redis 인스턴스의 현재 메모리에 대한 dump (스냅샷) 를 생성하는 기능
- RDB는 특정 시점의 메모리에 있는 데이터 전체를 바이너리 파일로 저장하는 것이다.
AOF 파일보다 사이즈가 작다. 따라서 로딩 속도가 AOF 보다 빠르다.
- 저장 시점을 정하는 redis.conf의 파라미터는 save 이다.
SAVE 조건은 여러 개를 지정할 수 있고, 모두 or 조건이다. 즉 어느 것 하나라도 만족하면 저장한다.
```
save 900 1   900초(15분) 동안 1번 이상 key 변경이 발생하면 저장 
save 300 10   300초(5분) 동안 10번 이상 key 변경이 발생하면 저장 
save 60 10000   60초(1분) 동안 10000번 이상 key 변경이 발생하면 저장 
```

### AOF (Append Only File)
- appendOnly.aof file 에 기록됨
- 명령이 실행될 때마다 해당 명령이 파일에 기록 (명령실행시 백업되는게 아닌 버퍼를 두고 주기적으로 파일에  쓴다)
- Rewrite 특정 시점에 데이터 전체를 다시 쓰는 기능
  - 파일이 너무 커지면 OS 파일 사이즈에 제한이 걸리거나, 레디스 시작시 AOF파일을 로드하는데 시간이 오래걸림
  - 같은 Key 를 100번 수정하는 경우 AOF 에는 100번 수정한 내용이 모두 기록되지만, rewrite 를 하게 되면 최종 수정된 마지막 값만 남기 때문입니다.
  - AOF는 text파일이어서 수정이 가능함

## cluster


----------------------------------------------------------------
Redis 유튜브 영상 정리 (https://www.youtube.com/watch?v=mPB2CZiAkKM&t=4634s)

Memcached는 컬렉션 제공안함, Redis는 제공함

Redis는 자료구조가 atomic하다 (race condition을 피할 수 있음)


## Redis 사용처
- Remote Data Store: A서버, B서버, C서버에서 데이터를 공유하고 싶을때
- 인증 토큰 저장 (Strings, Hash)
- Ranking 보드(Sorted Set)
- 유저 API Limit
- job queue (list)


### Sorted Sets (중요)
이 자료구조의 score는 double 타입이기에 값이 정확하지 않을 수 있음.
컴퓨터에서는 실수가 표현할 수 없는 정수값들이 존재


## Collection 주의사항
- 하나의 컬렉션에 너무 많은 아이템을 담으면 좋지 않음
  - 10000개 이하 몇천개 수준으로 유지하는게 좋음
- Expire는 Collection의 Item개별로 걸리지않고 전체 Collection에 대해서만 걸림
  - 즉 해당 10000개의 아이템을 가진 Collection에 expire가 걸려있다면 그 시간 후에는 10000개의 아이템이 모두 삭제


  ## Redis 운영
  - 메모리 관리를 잘하자
  - O(N) 관련 명령어는 주의하자
  - Replication
  - 권장 설정 Tip


### 메모리 관리
- Redis는 In-Memory Data Store
- Physical Memory 이상을 사용하면 문제가 발생
  - Swap이 있다면 Swap 사용으로 해당 메모리 page접근시 마다 늦어짐
- Maxmemory를 설정하더라도 이보다 더 사용할 가능성이 큼
- RSS 값을 모니터링 해야함

##### RSS: Resident Set Size이며 해당 프로세스에 할당되고 RAM에있는 메모리 양을 표시하는 데 사용

- 많은 업체가 현재 메모리를 사용해서 Swap을 쓰고 있다는것을 모를때가 많음
- 큰 메모리를 사용하는 instance 하나보다는 적은 메모리를 사용하는 instance 여러개가 안전함
- Redi는 메모리 파편화가 발생할 수 있음
  - 다양한 사이즈를 가지는 데이터 보다는 유사한 크기의 데이터를 가지는 경우가 메모리 파편화가 덜 일어남(더 효율적)

### 메모리가 부족할때는?
- 장비를 Migration
- 있는 데이터를 줄이자
  - 데이터를 일정 수준에서만 사용하도록 특정 데이터를 줄임
  - 이미 swap을 사용중이라면 프로세스를 재시작해야함

### 메모리를 줄이기 위한 설정
기본적으로 Collection들은 다음과 같이 사용
- Hash -> HashTablr을 하나 더 사용
- Sorted Set -> Skiplist와 HashTable을 이용
- Set -> HashTable 사용
- 해당 자료구조들은 메모리를 많이 사용

Ziplist를 이용하자(??)
- List, hash, sorted set 등을  ziplist로 대체해서 처리하는 설정이 존재함

- hash-max-ziplist-entries 설정(?? 자료구조별로 있는듯)
