# chapter 5. 분산 트랜잭션

## transaction

- 서비스 간 데이터 일관성을 유지하기 위해 여러 읽기, 쓰기를 논리적 단위로 그룹화 하는 방법
- 원자적으로 단일 작업으로 실행
- 전체 트랜잭션이 성공 → 커밋
- 전체 트랜잭션이 실패 → 중단 혹은 롤백
- ACID 속성을 갖지만 데이터베이스마다 다르므로 구현도 다르다.

## 분산 트랜잭션

- 별개의 쓰기 요청을 하나의 분산된(원자적) 트랜잭션으로 결합
- 합의 consensus
    - 모든 서비스가 쓰기 이벤트가 발생했거나 발생하지 않았다는 것에 동의하는 것
    - 서비스간 일관성을 위해 쓰기 이벤트 중 문제가 생기더라도 합의가 이뤄져야 함.

### 이벤트 기반 아키텍쳐 EDA event driven architecture

- 비동기적, 논블로킹 방식
- 요청 처리 대신 이벤트를 발행
    - 요청은 높은 지연을 초래할 수 있으므로 처리될 필요 없음.
    - 이벤트가 성공적으로 발행되면 성공 응답 반환, 이벤트는 그 후에 처리
- 느슨한 결합, 확장성, 응답성(낮은 지연시간)을 강화
- 대안
    - 서비스가 직접 다른 서비스에 요청
        - 요청 처리까지 가용 스레드가 줄어든다
        - 비싸고, 복잡하고 오류가 발생하기 쉽고 확장성이 떨어짐
    - 이벤트 로그에 이벤트를 발행 → 리소스가 적은 접근 방식

### 이벤트 소싱

- 추가 전용 로그에 이벤트로 데이터나 데이터 변경을 저장하는 패턴
- 클라우드 네이티브 패턴
    - 이벤트 로그가 신뢰할 수 있는 단일 데이터 출처이며 다른 모든 데이터베이스는 이벤트 로그에서 파생된 데이터 표현
- 모든 쓰기는 먼저 이벤트 로그에 이뤄져야 함.
- 특정 데이터 소스에 묶여있지 않음.
- 다양한 방식으로 구현할 수 있음
    - 카프카 토픽 등 이벤트 스토리지나 추가 전용 로그에 이벤트를 발행
    - RDBMS에 문서를 쓰거나
    - in-memory database에 쓸 수도 있음
- 모든 이벤트에 대한 완전한 감사 추적을 제공
- 디버깅, 분석 등을 위한 이벤트를 다시 실행해 시스템 과거 상태를 이해할 수 있게 해줌
- 새로운 이벤트 유형과 핸들러를 도입해 비지니스 로직을 변경할 수 있게 해줌
- 시스템 설계와 개발 복잡, 저장 요구사항이 증가

### CDC change data capture

- 데이터 변경 이벤트를 변경 로그 이벤트 스트림에 기록하고 이를 API를 통해 제공하는 방식
- 그림 5.2
- 이벤트 스트림에 각각 서비스/application/database 에 해당하는 여러 소비자가 있고 각 소비자는 이벤트를 소비하고 이를 다운스트림 서비스에 제공하여 처리
- 이벤트 소싱보다 일관성이 높고 지연시간이 낮음
- 요청을 거의 실시간으로 처리
- transaction log tailing pattern
    - 프로세스가 db에 쓰거나 카프카에 생성해야할 때 발생하는 불일치를 방지
    - 그림 5.3

### 이벤트 소싱과 cdc 비교

|  | 이벤트 소싱 | CDC |
| --- | --- | --- |
| 목적 | 이벤트 | 소스서비스에서 다운스트림 서비스로 이벤트를 전파해 데이터 변경을 동기화 |
| 기준 데이터 | 로그, 로그에 발행된 이벤트 | 발행자 서비스의 데이터베이스. 이벤트는 기준 데이터가 아님 |
| 세분성 | 특정 작업이나 상태 변경을 나타내는 세분화된 이벤트 | 새로 생성, 업데이트, 삭제된 행이나 문서 같은 개별 데이터베이스 수준의 변경 |

### transaction manager

- 트랜잭션이 성공적으로 완료되거나 취소되게 보장하는 프로세스
- 그림 5.4
- 불일치 수동 검토, 보상 트랜잭션의 수동실행을 위한 인터페이스로 먼저 구현되어야 함
- 보상 트랜잭션을 자동화하는 것은 위험, 항상 기록되어야 함

### Saga pattern

- 트랜잭션으로 작성할 수 있는 장기 실행 트랜잭션
- 모든 트랜잭션이 성공적으로 완료 되어야 함
- 롤백을 위한 보상 트랜잭션이 실행
- 상태 비저장
- 사용 사례
    - 특정 서비스가 특정 요구사항을 충족할 때만 분산 트랜잭션을 수행
        - 결제 서비스는 항공권 서비스가 티켓이 가용함을 확인, 호텔 객실 서비스 객실이 가용함을 확인할 때까지 어떤 결재도 처리해서는 안됨
- 코레오그래피 방식
    - 서비스는 두개의 카프카 토픽과 통신
        - 분산 트랜잭션을 시작하기 위해 카프카 토픽에서 생성
        - 최종 로직을 수행하기 위해 다른 카프카 토픽에서 소비
    - 그림 5.5
        1. 예약 서비스에 예약 요청 → 예약 토픽에 예약 요청 이벤트 생성
        2. 티켓 서비스와 호텔 서비스가 예약 요청 이벤트를 소비
            - 각 데이터베이스에 이 이벤트를 기록할 수 있음. awaiting_payment 같은 상태를 포함
        3. 티켓 서비스와 호텔 서비스는 각 티켓, 호텔 토픽에 결제 요청 이벤트 생성
        4. 결제 서비스는 티켓,호텔 토픽을 소비. 이 두 이벤트는 각기 다른 시간에 다른 호스트가 소비하므로 결제 서비스는 이 이벤트의 수신을 데이터베이스에 기록. 필요한 모든 이벤트가 수신되면 결제 처리
        5. 결제 성공하면 결제토픽에 결제 성공 이벤트 생성
        6. 티켓, 호텔, 예약 서비스가 이 이벤트를 소비 → 예약 서비스는 사용자에게 예약이 확인됐음을 알릴 수 있음.
        - 1~4 단계는 롤백할 수 있는 보상 가능한  트랜잭션
        - 5단계는 피벗 트랜잭션. 이후 트랜잭션은 성공할 때까지 재시도할 수 있음
        - 6단계는 재시도 가능한 트랜잭션, CDC.
    - 특징
        - 양방향 선이 없다. 서비스가 동일한 토픽에 생성/구독하지 않는다
        - 두 개의 서비스가 동일한 토픽에 생성하지 않음
        - 서비스는 여러 토픽을 구독할 수 있음. 특정 이벤트를 받았음을 db에 기록해야 함. → 이를 통해 모든 이벤트가 수신되었는지 확인 가능
        - 토픽:서비스는 1:N or N:1
        - 순환이 있을 수 있음
- 오케스트레이션 방식
    - 오케스트레이터
        - 사가 패턴을 시작하는 서비스가
        - 카프카 토픽을 통해 각 서비스와 통신
        - 이벤트에 반응하고 명령을 발행하는 유한 state machine
        - 순서 이외에 보상 메커니즘을 제외하고 다른 비지니스 로직을 포함하면 안됨
    - 그림 5.6
- 코레오그래피 vs 오케스트레이션
    
    
    | 코레오 | 오케스트레이션 |
    | --- | --- |
    | 서비스 요청이 병렬. 관찰자 객체 지향 | 서비스 요청이 선형적. 컨트롤러 객체 지향 |
    | 개발자는 사가 패턴과 관련된 모든 서비스의 코드를 읽어야 단계를 이해할 수 있음 | 오케스트레이터의 코드를 읽으면 분산 트랜잭션의 서비스와 단계를 이해할 수 있음 |
    | 여러 카프카 토픽을 구독해야 할 수도 있음 → 소비한 이벤트를 데이터베이스에 기록해야 함 | 각 서비스는 하나의 토픽만 구독. 데이터베이스 쓰기 횟수를 줄일 수 있음 |
    | 덜 자원 집약적, 통신량이 적음. 네트워크 트래픽이 적어 지연시간이 낮음 | 모든 단계가 오케스트레이터를 거쳐야 하므로 더 통신량도 많고 트래픽이 많아 지연시간 높음 |
    | 병렬 → 지연시간 낮음 | 선형 → 지연 시간 높음 |
    | 보상 트랜잭션은 사가 패턴과 관련된 다양한 서비스에 트리거 됨 | 보상 트랜잭션은 오케스트레이터에 의해 트리거 됨 |
- 그외 트랜잭션 유형
    - 정족수 쓰기
    - 팍소스 EPaxos
    - Raft
    - Zab(주키퍼 원자적 브로드캐스트 프로토콜)