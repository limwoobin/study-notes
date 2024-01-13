## Redis Cluster Spec

#### Redis 클러스터의 목표
- 최대 1,000 개의 노드까지 고성능 및 확장성을 제공, 프록시가 없고 비동기 복제가 사용됨
- Redis Cluster 는 특정 key를 동일한 해시 슬롯에 저장하기 위해 사용되는 Hash Tags 라는 개념을 구현함

#### Redis Clustrer 란
- 여러 Redis 서버에 자동으로 샤딩을 해주는 기술
- Redis Cluster 는 2개의 TCP 연결이 열려있어야 함
- 모든 Redis Cluster 의 노드들은 기본포트(6379)와 기본포트 + 10000 (16379) 같은 두개의 포트를 사용
- 16379 와 같은 포트는 레디스 클러스터 버스로 사용됨 
이 레디스 클러스터 버스는 장애 감지, 구성 업데이트, failover 승인 등에 사용됨, 따라서 반드시 두개의 포트는 통신을 보장해주어야함

#### Redis Cluster Data Sharding
- Redis Cluster 는 hash slot 이라는 개념을 사용함, 이 hash slot 은 16384 개의 hash slot 이 존재
- 데이터 저장시 주어진 키의 hash slot 을 찾기 위해 `HASH_SLOT=CRC16(Key) mod 16384` 와 같은 알고리즘 사용
	- 일반적인 모듈러연산은 노드가 추가/제거됨에 따라 기존 노드의 데이터들에 대해 다시 연산되고, 이동되어야 하는 데이터가 많은데
	  이에 반해, 해당 알고리즘은 겹치는 데이터들에 대해서만 효율적으로 이동하면 됨
- 만약 Redis Cluster 를 3개의 노드로 구성한다면 다음과 같이 hash slot 이 구성됨
	- Node A contains hash slots from 0 to 5500.
	- Node B contains hash slots from 5501 to 11000.
	- Node C contains hash slots from 11001 to 16383.
- 특정 노드를 제거하려면 해당 노드의 hash slot 이 완전 비어있어야 제거 가능


#### 데이터 일관성
- 레디스는 `String Consistency` 를 제공하지 않음.
- 즉, 데이터 저장 시 복제가 완료된 이후 응답을 보내는것이 아닌 성능을 위해 응답과 동시에 비동기로 replication 을 수행함
- 그렇기에 복제가 완료되기 전에 master 가 죽으면 데이터가 유실될 수 있음
	- 1. 클라이언트가 마스터 노드에 데이터 입력
	- 2. 마스터 노드는 클라이언트에게 ACK 응답을 보냄
	- 3. 마스터 노드는 비동기로 레플리카 노드인 B1 에 입력받은 데이터를 저장

#### Redis Cluster master-slave
- Redis Cluster 는 hash slot 에 1개~N개의 복제본이 있는 master-slave 모델을 사용한다
- 마스터 노드인 A, B, C 그리고 슬레이브 노드인 A1, B1, C1 으로 구성될 수 있음
- 이런 경우 노드 B 가 죽어도 B1 이 승격해서 일을 진행하기에 시스템을 계속 사용할 수 있음
- 하지만 B, B1 이 모두 죽는 경우엔 클러스터를 계속 사용할 수 없음


#### 제약사항
- Redis Cluster 에서는 MGET, MSET 과 같은 멀티키 연산이 불가능하다
	- 키들이 각기다른 hash slot 에 저장되어 있기에 그렇다 
	- `CROSSSLOT Keys in request don't hash to the same slot` 에러 메시지 발생
- 클러스터 환경멀티키 연산을 하고 싶다면 hashtag 라는 개념을 이용하여 데이터 저장시 같은 hash slot 에 저장해야함
e.g.
```
127.0.0.1:30001> set {city}:1000 seoul {city}:2000 tokyo {city}:3000 newyork
OK
```
- 이렇게 저장하면 redis 는 `{}` 사이의 값을 가지고만 해싱하기에 같은 hash slot 에 데이터를 저장할 수 있음

- Redis Cluster 에서는 database 사용시 `DB 0` 만 사용할 수 있음
	- Redis 의 database 는 mysql 의 schema 와 유사한 개념
	- 싱글 Redis 는 인스턴스에 여러 database 를 가질 수 있음
