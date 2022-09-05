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

## AOF

## cluster
