# chapter 4 분산 메세지 큐

- 메세지 큐는 잘 정의된 인터페이스를 경계로 나뉜 작고 독립적인 블록들 사이의 통신과 조율을 담당하며 다음과 같은 이득을 얻을 수 있다
    - 컴포넌트 간 결합도 완화 decoupling
    - 규모 확장성 개선
        - consumer 의 시스템 규모를 트래픽 부하에 맞게 늘리거나 하여 처리 용량 늘릴 수 있다
    - 가용성 개선
        - 특정 컴포넌트에 장애가 발생해도 다른 컴포넌트는 큐와 계속 상호작용 할 수 있다.
    - 성능 개선
        - 메세지 큐를 이용하면 비동기 통신이 쉽게 가능
- 유명 분산 메세지 큐
    - [apache Kafka](https://kafka.apache.org/)
    - [apache RocketMQ](https://rocketmq.apache.org/)
    - [apache rabbitMQ](https://rabbitmq.com/)
    - [apache Pulsar](https://pulsar.apache.org/)
    - [apache ActiveMQ](https://activemq.apache.org/)
    - [ZeroMQ](https://zeromq.org/)
- message queue vs event streaming platform
    - 지원하는 기능이 서로 수렴하며 차이가 점차 희미해지고 있음
    - event streaming platform
        - apache kafka, pulsar
        - pulsar는 kafka와 경쟁관계이나 분산 메세지 큐로도 사용 가능할 정도로 유연하고 성능이 좋음
    - message queue
        - rocketMQ, ActiveMQ, RabbitMQ, zeroMQ

## 1단계: 문제 이해 및 설계범위 확정

- MQ의 기본 기능 → producer는 message를 Queue에 보내고 consumer는 queue에서 message를 꺼낼 수 있으면 된다.

### 기능 요구사항

- 생산자는 메세지 큐에 메세지를 보낼 수 있어야 한다
- consumer는 MQ를 통해 message를 수신할 수 있어야 한다
- message는 반복 수신/단 한번 수신 등의 설정이 가능해야 함
- 오래된 이력 데이터는 삭제될 수 있다
- 메세지 크기는 kb 수준이다
- 메세지는 생산된 순서대로 consumer에게 전달될 수 있어야 한다
- message 전달 방식은 최소한번, 최대 한번, 정확히 한번 중 설정할 수 있다

### 비기능 요구사항

- 높은 대역폭 vs 낮은 전송 지연 중 하나를 설정으로 선택
- 규모 확장성: 메세지 양이 급증해도 처리 가능해야 함
- 지속성persistency 내구성 durablility
    - 데이터는 디스크에 지속적으로 보관되어야 하며 여러 노드에 복제되야 함

### 전통적 메세지 큐와 다른 점

- RabbitMQ 등은 메세지가 consumer에게 전달되기 충분한 기간동안만 메모리에 보관
- 전달 순서도 보존하지 않는다.

## 2단계: 개략적 설계안 제시 및 동의 구하기

- 메세지 큐의 기본 기능
    - 그림 4.2
    - producer는 message를 queue에 발행
    - consumer는 queue 를 subscribe하고 구독한 메세지를 소비
    - MQ는 Producer와 Consumer사이 결합을 느슨하게 하는 서비스. 각각 독립적으로 운영/ 규모확장 가능하게 하는 역할을 담당한다
    - client/server model 관점에서 producer, consumer 모두 client이고 mq는 server 역할을 하며 client와 server는 network를 통해 통신한다

### 메세지 모델

- 일대일 모델point-to-point
    - 전통적 MQ에서 흔히 볼 수 있음.
    - 큐에 전송된 메세지는 소비자가 아무리 많더라도 오직 한 소비자만 가져갈 수 있다
    - 그림 4.3
    - consumer가 메세지를 가져갔다는 사실을 queue 에 acknowledge하면 메세지는 큐에서 삭제
        - 본 설계안은 메세지를 두 주동안은 보관할 수 있도록 persistence layer를 포함하여 메세지가 반복적으로 소비될 수 있도록 할 것
- 발행/구독 모델publish-subscribe
    - topic
        - 메세지를 주고 받을 때는 토픽에 보내고 받게 된다.
        - 메세지 큐 서비스 전반에 고유한 이름.
        - 그림 4.4

### 토픽, 파티션, 브로커

- 메세지는 토픽에 보관된다
- 파티션
    - sharding 기법. 토픽에 보관되는 데이터의 양이 커지면.. 토픽을 여러 파티션으로 분할한 다음 메세지를 모든 파티션에 균등하게 나눠 보낸다.
    - 그림 4.5
    - 파티션은 토픽에 보낼 메세지의 작은 부분 집함
    - 메세지 큐 클러스터 내 서버에 고르게 분산 배치.
        - 각 파티션은 FIFO 큐처럼 동작
    - 토픽을 구독하는 consumer는 하나 이상의 파티션에서 데이터를 가져오게 된다.
    - 토픽을 구독하는 소비자가 여럿인 경우 각 consumer는 해당 토픽을 구성하는 파티션의 일부를 담당
- 브로커
    - 파티션을 유지하는 서버
    - 파티션을 브로커에 분산하는 것이 높은 규모 확장성을 달성하는 비결
    - 그림 4.6

### consumer group

- 본 설계안은 point-to-point, pub-sub 두 가지 모델 전부를 지원해야 함
- consumer group 내 consumer는 토픽에서 메세지를 소비하기 위해 서로 협력
- 하나의 소비자 그룹은 여러 토픽을 구독할 수 있고 오프셋을 별도로 관리한다
- 그림 4.7
    - 데이터를 병렬로 읽으면 throughput 측면에서는 좋지만
    - 소비자 1, 소비자 2가 같은 파티션 1 의 메세지를 읽는 경우 메세지 소비 순서를 보장할 수 없다.
    - 어떤 파티션의 메세지는 한 그룹 안에서 오직 한 소비자만 읽을수 있도록 제약을 넣으면 되지만
    - 그룹내 소비자 수가 파티션 수보다 크면 어떤 consumer는 토픽에서 데이터를 읽지 못하게 됨.

### 개략적 설계안

- 그림 4.8
- 클라이언트
    - producer: message를 특정 토픽으로 보낸다
    - consumer: 토픽을 구독하고 메세지를 소비한다
- 핵심 서비스 및 저장소
    - 브로커
        - 파티션을 유지
        - 하나의 파티션은 특정 토픽에 대한 메세지의 부분 집합을 유지
    - 저장소
        - 데이터 저장소: 메세지는 파티션 내 데이터 저장소에 보관
        - 상태 저장소: consumer 상태 저장
        - 메타데이타 저장소: 토픽설정, 토픽 속성 등
    - 조정 서비스 coordination service
        - service discovery: 어떤 브로커가 살아 있는지 알려준다
        - 리더 선출 leader election
            - 브로커 가운데 하나는 controller 역할
            - 한 클러스터 안에 반드시 활성상태 컨트롤러가 하나 있어야 하고, 이 컨트롤러가 파티션 배치를 책임진다
        - 아파치 주키퍼 또는 etcd가 보통 컨트롤러 선출을 담당하는 컴포넌트

## 3단계: 상세 셜계

- 디스크 기반 자료 구조on-disk data structure를 활용
    - rotational disk의 높은 순차 탐색 성능, 적극적 디스크 캐시 전략 aggressive disk caching strategy 을 잘 이용

### 데이터 저장소

- 특징
    - 읽기와 쓰기가 빈번히 일어나고
    - 갱신/삭제 연산은 발생하지 않고
    - 순차적 읽기/쓰기가 대부분
- 선택지1: 데이터베이스
    - RDBMS: 토픽별 테이블을 생성
    - NoSQL: 토픽별 collection을 만든다
    - I/O가 빈번히 발생하는 연산을 잘 처리하는 데이터베이스는 설계하기 어려워서 no
- 선택지2: 쓰기 우선 로그 Write-Ahdead Log, WAL
    - 새로운 항목이 추가되기만 하는 append-only 일반 파일
    - 지속성을 보장해야 하는 메세지는 디스크에 WAL로 보관할 것을 추천
    - WAL 접근 패턴은 읽기/쓰기 전부 순차적이고, 접근 패턴이 순차적일 때 디스크는 아주 좋은 성능을 보임
    - 그림 4.9 새로운 메세지는 파티션 끝에 추가되며 오프셋은 점진적으로 증가함.
        - 로그 파일의 line number를 오프셋으로 사용할 수 있다
    - 세그먼트
        - 파일 크기를 무한정으로 늘릴 수 없으므로 세그먼트 단위로 나누는 것이 바람직
        - 일정 한계에 도달하면 새 활성 새그먼트를 만들어 새 메세지를 수용하고 좀전까지 활성 상태였던 세그먼트에는 비활성화 하고 읽기 요청만 처리
        - 보관 기한이 만료되거나 용량한계에 도달하면 삭제

### 메세지 자료 구조

- 메세지 구조는 높은 대역폭 달성의 열쇠
- 생산자, 메세지 큐, 소비자 사이의 계약이다


    | 필드 이름 | 데이터 자료형 |  |
    | --- | --- | --- |
    | key | byte[] | 파티션을 정할 때 사용.
    키가 있는 경우 hasy(key) % numPartitions의 공식에 따라 결정, 없으면 무작위 |
    | value | byte[] | 메세지의 내용, payload |
    | topic | string | 메세지가 속한 토픽의 이름 |
    | partition | integer | 메세지가 속한 파티션의 id |
    | offset | long | 파티션 내 메세지의 위치. 메세지는 토픽, 파티션, 오프셋 세가지 정보를 알면 찾을 수 있다 |
    | timestamp | long | 메세지가 저장된 시간 |
    | size | integer | 메세지 사이즈 |
    | crc | integer | crc cyclic redundancy check 순환 중복 검사. 주어진 데이터이ㅡ 무결성을 보장하는데 이용 |

### 일괄 처리

- batching. 성능 개선에 중요하다
- 값비싼 네트워크 왕복 비용을 제거
- 더 높은 디스크 접근 대욕폭을 달성할 수 있다.

### producer 작업 흐름

- 그림 4.11
    1. message를 routing layer로 전송
    2. 라우팅 계층은 replica distribution plan 사본 분산 계획을 읽어 자기 캐시에 보관
    3. 메세지가 도착하면 리더 사본에 보낸다.
    4. 리더 사본이 메세지를 받고 해당 리더를 따르는 다른 replica는 리더로부터 데이터를 받는다.
    5. 충분한 수의 사본이 동기화 되면 leader는 데이터를 디스크에 commit
- 라우팅 계층을 도입하면 거쳐야 할 네트워크 노드가 하나 더 늘어나 오버해드가 발생
- 4.12 수정
    - 라우팅 계층을 producer 내부로 편입하고 버퍼를 도입
    - 네트워크를 거칠 필요가 줄어들어 전송 지연도 줄어든다
    - 생산자는 메세지를 어느 파니션에 보낼지 결정하는 자신만의 로직을 가질 수 있다.
    - 전송할 메세지를 버퍼 메모리에 보관했다가 목적지로 일괄 전송하여 대역폭을 높일 수 있다.

### consumer측 작업 흐름

- push 모델
    - 브로커가 데이터를 consumer에게 전송
    - 장점: 낮은 지연, 메세지를 받는 즉시 consumer에 보낼 수 있다
    - 단점
        - consumer가 메세지 소비하는 속도가 생산하는 속도보다 느리면 과부하 걸릴 수 있다.
- poll 모델
    - 장점
        - 메세지 소비하는 속도를 소비자가 알아서 결정
        - 메세지 소비하는 속도가 딸리면 소비자를 늘려서 해결하거나 생산 속도를 따라잡을 때까지 기다려도 됨.
        - 일괄처리에 적합하다.
    - 단점
        - 메세지가 없어도 계속 끌고가려고 하여 자원 낭비
    - 동작 흐름도 4.15
        1. 새로운 consumer가 그룹 1에 합류하고 토픽 a를 구독하길 원하는  그룹 이름을 hashing하여 접속할 broker node를 찾는다.
            1. 같은 consumer group의 consumer는 같은 broker에 접속하며 이때 브로커를 coordinator 라고 부른다.
        2. coordinator 는 새 consumer를 그룹에 참여시키고 partition-2를 새 consumer에 할당한다.
            1. 파티션 배치 정책은 round-robin 이나 range 기반 정책 등 여러가지가 있다
        3. consumer는 마지막으로 소비한 offset 이후의 메세지를 가지고 온다.
            1. 오프셋 정보는 state storage에 저장
        4. consumer는 메세지를 처리하고 새로운 offset을 broker에 보낸다.

### 소비자 재조정 consumer rebalancing

- 어떤 consumer가 어떤 partition을 책임지는지 다시 정하는 프로세스
- 새로운 consumer가 합류, 그룹을 떠나거나 장애가 발생할 때, 파티션이 조정되는 경우에 시작됨
- 코디네이터
    - consumer rebalancing을 위해 consumer들과 통신하는 broker node
    - consumer 의 heartbeat를 체크하고 각 consumer의 partition 내 offset 정보를 관리
- 코디네이터는 consumer의 목록을 유지하고 이 list에 변화가 생기면 해당 consumer group의 leader를 새로 선출
- 새 리더는 새 partition dispatch plan 파티션 배치 계획을 만들고 코디네이터에 전달하고 코디네이터는 이를 그룹 내 모든 consumer에 전달한다.

  > 재조정 시나리오1: 새로운 consumer의 합류
  >
  > 1. 새 consumer가 합류를 요청하면
  > 2. coordinator는 재조정이 필요하다고 판단하고 모든 consumer에게 수동으로 그룹에 재합류하라고 통지한다.
  > 3. 모든 consumer가 합류하고 나면 coordinator는 그 가운데 하나를 leader로 선출한다
  > 4. leader는 partition 배치 계획을 생성 후 코디네이터에 전달하고 leader 외 consumer는 코디네이터에게 요청하여 파티션 배치 계획을 받아온다.
  > 5. consumer는 자기에게 배치된 partition에서 message를 가져오기 시작한다.
  >
  > 재조정 시나리오2: consumer가 group을 떠나는 경우
  >
  > - 그림 4.19
  > - 그룹 탈퇴를 요청하면 그 다음부터는 위의 2 이후와 동일하다
  >
  > 재조정 시나리오3: consumer 중 하나에 장애가 나면
  >
  > - 그림 4.20
  > - heartbeat가 더이상 코디네이터에 전달되지 않으면 해당 consumer가 사라진 것으로 판단하여 재조정 프로세스를 개시
  > - 그 이후는 위 2부터와 동일하다.

### 상태 저장소

아래의 정보가 저장

- consumer에 대한 파티션의 배치 관계
- 각 consumer group이 partition에서 마지막으로 가져간 메세지의 offset



### 작성 중..ㅠ

오늘 수업중에

- [https://bistros.tistory.com/entry/Kafka-idempotent-producer-멱등성에-관해서](https://bistros.tistory.com/entry/Kafka-idempotent-producer-%EB%A9%B1%EB%93%B1%EC%84%B1%EC%97%90-%EA%B4%80%ED%95%B4%EC%84%9C)
- [https://velog.io/@eastperson/Transaction-Outbox-Pattern-알아보기](https://velog.io/@eastperson/Transaction-Outbox-Pattern-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)
- exactly at once를 해보려면
    - produer에서는 idempotent 를 true 주고
        - broker에서 같은 메세지 키인 경우 씹어버림.
    - consumer 에서는 중복 메세지를 따로 처리하는 로직을 넣으면