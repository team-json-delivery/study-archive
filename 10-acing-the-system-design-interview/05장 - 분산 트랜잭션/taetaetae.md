## 05. 분산 트랜잭션
- 트랜잭션은 서비스 간 데이터 일관성을 유지하기 위해 여러 읽기와 쓰기를 논리적 단위로 그룹화하는 방법
- 트랜잭션은 원자적으로 단일 작업으로 실행되며, 전체 트랜잭션이 성공해 커밋하거나 실패해 중단 혹은 롤백

#### 5.1 이벤트 기반 아키텍처(EDA)
- 다른 구성 요소 간의 대부분의 상호 작용이 작업 수행을 요청하는 대신 이미 발생한 이벤트를 알림으로써 실현 되는 아키텍처 스타일
- 비동기적이고 논블로킹(Non-blocking)
- 느슨한 결합, 확장성, 그리고 응답성(낮은 지연 시간)을 강화
- 강력한 일관성과 낮은 지연시간이 필요하지 않을 경우 적용

#### 5.2 이벤트 소싱
- 추가 전용 로그에 이벤트로 데이터나 데이터 변경을 저장하는 패턴
- 모든 쓰기는 먼저 이벤트 로그에 저장 후 진행
- 특정 데이터 소스에 묶여있지 않고 다양한 소스에서 이벤트를 캡처 가능
- 질문
  1. 구독자 서비스가 이벤트를 처리하다 실패하면?
    at-least-once delivery
    이벤트는 최소 한 번 이상 전달 → 중복 처리에 대비
  2. 구독자는 이벤트를 다시 처리해야 한다는 것을 어떻게 알까?
  	Kafka - 처리 이후 commit offset을 업데이트, 업데이트 되지 않은 경우 재 전송
  	RabbitMQ - 일정 시간 내 처리가 안되는 경우 다시 큐에 들어가고 별도로 재처리 필요

  3. 재처리 전략과 충돌 대응 설계
  	멱등성 보장, DLQ 활용
- 완전한 감사 추적 제공, 시스템의 과거 상태 재현 가능. but, 비용

#### 5.3 변경 데이터 캡처
- 데이터 변경 이벤트를 변경 로그 이벤트 스트림에 기록하고 이 이벤트 스트림을 API를 통해 제공
- 이벤트 소싱보다 일관성이 높고 지연시간이 낮음(거의 실시간)
- 트랜잭션 로그 테일링 패턴 : 서비스가 데이터베이스에 쓰기 쿼리를 수행하면 데이터베이스는 이 쿼리를 로그 파일에 기록. 로그 마이너는 로그 파일을 테일링 해서 메시지 브로커에 이벤트 생성
- 트랜잭션 로그 마이너는 중복 이벤트 생성 가능 (정확히 한 번 전달 or 멱등하게 처리 로 중복처리 해소)

#### 5.4 이벤트 소싱과 CDC 비교

| | 이벤트 소싱 | 변경 데이터 캡쳐(CDC) |
| --- | --- | --- |
| 목적 | 이벤트를 기준으로 기록 | 소스 서비스에서 다운 스트림 서비스로 이벤트를 전파해 데이터 변경을 동기화 |
| 기준 데이터 | 로그나 로그에 발행된 이벤트 | 발행자 서비스의 데이터 베이스 |
| 세분성 | 특정 작업이나 상태 변경을 나타네는 세분화된 이벤트 | 개벌 데이터 베이스 수준의 변경 |

#### 5.5 트랜잭션 감독자
- 트랜잭션이 성공적으로 완료되거나 취소되게 보장하는 프로세스
- 주기적인 배치나 서버리스 함수로 구현 가능
- 보상 트랜잭션을 자동화 할때는 일반적으로 위험하며 주의를 기울여 접근 필요 (자동이든 수동이든 항상 기록 필요)

#### 5.6 사가 패턴
- 장기 실행 트랜잭션, 실패시 보상 트랜잭션 실행, 상태 비저장
- Kafka 나 RabbitMQ 같은 메시지 브로커 활용
- 코레오그래피(병렬)
  - 메시지를 분류하고 저장하는 두개의 카프카 토픽과 통신
  - 보상 가능 트랜잭션 : 보상 트랜잭션으로 롤백 가능한 트랜잭션
  - 피벗 트랜잭션 : 사가의 진행/중단 지점. 피봇 트랜잭션이 커밋되면 사가는 완료될 때까지 실행. 피봇 트랜잭션은 보상 가능 트랜잭션, 재시도 가능한 트랜잭션 그 어느 쪽도 아니지만, 최종 보상 가능 트랜잭션 또는 최초 재시도 가능 트랜잭션이 될 수 있음
  재시도 가능 트랜잭션 : 피봇 트랜잭션 직후의 트랜잭션. 반드시 성공
- 오케스트레이션(선형)
  - 사가 패턴을 시작하는 서비스가 오케스트레이터
  - 카프카 토픽을 통해 각 서비스와 통신, 각 단계의 결과를 다른 토픽으로 소비
- 비교
  - 코레오그래피 : 서비스끼리 이벤트를 주고받으며 자율적으로 동작
    - 느슨한 결합(의존 낮음), 확장 용이
    - 흐름 파악 어려움, 복잡한 경우 난이도 증가
  - 오케스트레이션 : 중앙에서 지휘자처럼 전체 흐름을 통제
    - 흐름 명확, 중앙 제어
    - 단일 장애점, 결합도 증가

#### 5.7 다른 트랜잭션 유형
- 합의(Consensus) 알고리즘은 일반적으로 분산 데이터베이스에서 많은 수의 노드 합의를 달성하는데 유용
- 정족수 쓰기, 팍소스와 EPaxos, Ratt, Zab (주키퍼에서 사용)