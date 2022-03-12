# **Kafka**

## **Topic**

- 파티션은 늘릴 수는 있지만 다시 줄일 수는 없음
- 파티션 내의 데이터는 컨슈머가 컨슘한다해도 즉시 삭제되지 않음
- 동일 데이터에 대해선 아래의 경우 다른 컨슈머에 의해 두번 처리될 수 있음
  - 컨슈머 그룹이 다른 경우
  - auto.offset.reset = earliest 의 경우
- 삭제는 옵션에 따라 결정됨  
  log.retentions.ms: 최대 record 보존 시간  
  log.retention.byte: 최대 record 보존 크기 (byte)

<br>

## **Replication / broker**

- **파티션의 복제본을 저장하는곳**
  - replication 이 2라면 leader partion 1개 , follow partion 1개
  - replication 이 3이라면 leader partion - 1 , follow partion - 2

![kafka-image-1](https://user-images.githubusercontent.com/28802545/158009223-9c4b3c30-cbd7-4b0e-9928-1a837c91b473.PNG)

- **ack option**
  - **ack 0**
    - producer 는 leader partion 에 data 를 전송하고 응답값을 받지 않음
    - 그렇기 때문에 leader partion에 데이터가 정상적으로 전송되었는지 나머지 partion에 정상적으로 복제되었는지 보장할 수 없음
    - 속도는 빠르지만 데이터 유실 가능성이 있음
  - **ack 1**
    - producer 는 leader partion 에 data 를 전송하고 응답값을 받음
    - 다만 나머지 partion에 복제되었는지는 알 수 없음
    - 데이터 유실 가능성 존재
  - **ack all**
    - producer 는 leader partion 에 data 를 전송하고 응답값을 받음
    - 나머지 partion에도 data가 저장되는 것까지 확인함
    - 데이터 유실은 없는대신 0,1 에 비해 확인과정이 많기때문에 속도가 느림

<br>

- **파티셔너(Partitioner): 프로듀서가 data를 보내면 파티셔너를 통해 브로커로 데이터가 전달됨**

  - **메시지 키가 있는경우**

    - 메시지키를 가진 레코드는 파티셔너에 의해 특정한 hash 값이 생성됨, 이 hash값을 기준으로 어느 파티션에 들어갈지 결정됨
    - 동일한 키를 가진 레코드는 동일한 파티션으로 순차적으로 들어가게됨 (queue 처럼 동작)

  - **메시지 키가 없는경우**
    - round-robin 방식으로 파티션에 들어가게됨
    - 그런데 일반적인 round-robin 과는 약간 다르게 `UniformStickyPartitioner` 이라는 파티셔너가 프로듀서에서 배치로 모을수 있는 최대한의 레코드를 모아 파티션으로 데이터를 전송 (파티셔너를 따로 설정하지 않은 경우 default 가 `UniformStickyPartitioner`)
    - custom 파티셔너를 사용할 수 있게 파티셔너 interface 제공
