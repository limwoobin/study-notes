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

### AOF (Append Only File)
- appendOnly.aof file 에 기록됨
- 명령이 실행될 때마다 해당 명령이 파일에 기록 (명령실행시 백업되는게 아닌 버퍼를 두고 주기적으로 파일에  쓴다)
- Rewrite 특정 시점에 데이터 전체를 다시 쓰는 기능
  - 파일이 너무 커지면 OS 파일 사이즈에 제한이 걸리거나, 레디스 시작시 AOF파일을 로드하는데 시간이 오래걸림
  - 같은 Key 를 100번 수정하는 경우 AOF 에는 100번 수정한 내용이 모두 기록되지만, rewrite 를 하게 되면 최종 수정된 마지막 값만 남기 때문입니다.
  - AOF는 text파일이어서 수정이 가능함

## cluster
