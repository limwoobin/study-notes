# 카프카 컨슈머 전략

https://d2.naver.com/helloworld/0974525

- 리밸런스가 발생한 컨슈머 그룹은 리밸런스 작업이 끝날 때까지 브로커로부터 데이터를 가져오지 않는다. 따라서 리밸런스가 진행되는 동안에는 컨슈머들이 일시 정지된 것처럼 보인다.


- KafkaConsumer는 auto.offset.reset 설정에 따라 오프셋을 초기화한다.
earliest: 파티션의 가장 처음 오프셋을 사용한다.
latest: 파티션의 가장 마지막 오프셋을 사용한다.
none: 오프셋을 초기화하지 않는다.

- 만약 enable.auto.commit 설정이 true인 경우 KafkaConsumer가 auto.commit.interval.ms마다 오프셋을 자동으로 커밋한다. enable.auto.commit의 기본값은 true이고 auto.commit.interval.ms의 기본값은 5000ms(5초)이다.

- commitsync vs ack
https://stackoverflow.com/questions/59864610/kafka-acknowledgment-vs-kafka-commit

- Heartbeat 는 백그라운드 스레드로 동작한다

- max.poll.interval.ms
	- poll 메서드는 max.poll.interval.ms 이내에 호출되어야 한다.
	- 만약 poll 메서드가 제한 시간 내에 호출되지 않으면 해당 컨슈머는 정상이 아닌 것으로 간주되어 컨슈머 그룹에서 제외되고 컨슈머 리밸런스가 일어난다.
	- HeartBeat 스레드가 현재 시간과 마지막으로 poll 메서드가 호출된 시간의 차이를 계산하여 해당 시간의 차가 max.poll.interval.ms보다 큰 경우 컨슈머 그룹에서 탈퇴한다.
	- max.poll.interval.ms의 기본값은 300000ms(5분)이다.
- heartbeat.interval.ms
	- Heartbeat 전송 시간 간격이다. HeartBeat 스레드는 heartbeat.interval.ms 간격으로 Heartbeat을 전송한다.
	- heartbeat.interval.ms의 값은 항상 session.timeout.ms보다 작아야 하며 일반적으로 session.timeout.ms의 1/3 이하로 설정한다.
	- heartbeat.interval.ms의 기본값은 3000ms(3초)이다.
- session.timeout.ms
	- session.timeout.ms 내에 HeartBeat이 도착하지 않으면 브로커는 해당 컨슈머를 그룹에서 제거한다.
	- session.timeout.ms는 브로커 설정인 group.min.session.timeout.ms와 group.max.session.timeout.ms 사이 값이여야 한다. group.min.session.timeout.ms의 기본값은 6000ms(6초)이고, group.max.session.timeout.ms의 기본값은 버전에 따라 다른데 0.10.2에서는 300000ms(5분)이고 최신 버전(2.7)에서는 1800000ms(30분)이다.
	- session.timeout.ms의 기본값은 10000ms(10초)이다.


# Fetcher

- 브로커로부터 이미 가져온 데이터가 있는 경우에는 max.poll.records 설정 값만큼 레코드를 반환한다. max.poll.records의 기본값은 500이다.

- fetch.max.wait.ms
	- 브로커가 Fetch API 요청을 받았을 때 fetch.min.bytes 값만큼 데이터가 없는 경우 응답을 주기까지 최대로 기다릴 시간이다.
	- 기본값은 500ms(0.5초)이다.
- fetch.min.bytes
	- Fetch API 요청이 왔을 때 브로커는 최소한 fetch.min.bytes 값만큼 데이터를 반환해야 한다.
	- 반환할 만큼 데이터가 충분하지 않다면 브로커는 데이터가 누적되길 기다린다.
	- 기본값은 1이다.
- fetch.max.bytes
	- Fetch API 요청에 대해 브로커가 반환해야 하는 최대 데이터 크기이다.
	- 이 값은 절대적으로 적용되는 값은 아니다. 첫 번째 파티션의 첫 번째 메시지가 이 값보다 크다면 컨슈머가 계속 진행될 수 있도록 데이터가 반환된다.
	- 브로커가 허용하는 최대 메시지 크기는 message.max.bytes와 max.message.bytes를 통해 설정한다.
	- 기본값은 52428800(50MiB)이다.
- max.partition.fetch.bytes
	- 브로커가 반환할 파티션당 최대 데이터 크기이다.
	- fetch.max.bytes와 동일하게 첫 번째 파티션의 첫 번째 메시지가 이 값보다 크다면 컨슈머가 계속 진행될 수 있도록 데이터가 반환된다.
	- 기본값은 1048576(1MiB)이다.

KafkaConsumer가 올바르게 동작하기 위해서는 리밸런스는 필요하지만 안정적인 데이터 처리를 위해서 불필요한 리밸런스는 줄이는 것이 좋다. 불필요한 리밸런스를 줄이기 위해서는 max.poll.interval.ms와 max.poll.records를 적절히 조정하여 poll 메서드가 일정 간격으로 호출되도록 해야 한다. 필요한 경우에는 heartbeat.interval.ms와 session.timeout.ms를 조정한다.

앞에서 언급했듯이 리밸런스가 유발하는 Stop the world 현상을 완화시키기 위해 최근에 Kafka는 컨슈머 리밸런스 과정을 증분으로 진행하는 기능을 추가했다. 증분 리밸런스를 통해 리밸런스 과정에서 발생하는 컨슈머 정지 시간을 줄일 수 있다. 최신 버전의 KafkaConsumer를 사용한다면 증분 리밸런스 기능 사용을 고려해보면 좋을 것 같다.

-> 리밸런싱 cooperative 전략 (이게 증분전략, stop the world 시간이 적다)



# ----------------------------------------