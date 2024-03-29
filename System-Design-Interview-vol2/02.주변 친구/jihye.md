# 2. 주변친구 nearly friends

- 이전 챕터의 근접성 서비스의 사업장 주소는 정적이었지만 주변 친구의 위치는 자주 바뀔 수 있다는 큰 차이점이 있다.

## 1단계: 문제 이해 및 설계범위 확정

- 지리적으로 주변이란? 직선거리 5mile
- 얼마나 많은 사용자? 10억(1billion)을 가정하고 그 가운데 10%정도가 이 기능을 활용
- 이동 이력 보관 유무? 머신 러닝 등 다양한 용도로 사용하기 위해 저장
- 비활성 상태의 처리? 10분 이상 비활성 시 주변 친구 목록에서 사라지도록 처리
- GDPR(General Data Protection Regulation)이나 CCPA(California Consumer Privacy Act) 같은 사생활 보호법 고려? 일단은 고려하지 않음

### 기능 요구사항

- 모바일 앱에서 주변친구를 확인
- 주변친구 목록에 보이는 항목
    - 해당 친구까지의 거리
    - 그 정보가 마지막으로 갱신된 시각(timestamp)
- 친구 목록은 몇 초 마다 한 번씩 갱신

### 비기능 요구사항

- low latency빠르게 친구 위치 변화 반영
- 안정성 but 몇 개 데이터가 유실되는 정도는 용인
- 결과적 일관성 eventual consistency
    - 위치 데이터를 저장하기 위해 강한 일관성 strong consistency를 지원하는 데이터 저장소를 사용할 필요는 없고, 복제본의 데이터가 원본과 동일하게 변경되기까지 몇 초 정도 걸리는 것 가능

### 개략적 규모 추정

- 주변 친구는 5mile(8km) 반경 이내
- 걷는 속도(4~6km/h)가 느리기 때문에 위치 정보는 30초 주기로 갱신
- 평균적으로 매일 1억명의 사용자가 있고
- 동시접속자의 수는 DAU의 10%로 가정 → 천만
- 평균적으로 400명의 친구를 가진다
- 페이지당 20명의 주변 친구를 표시하고 사용자 요청시 더 많이 노출

> QPS query per second 계산
위치정보 갱신 QPS = 천만(동시접속사용자) / 30 (30초마다 갱신) =~ 334,000
>

## 2단계: 개략적 설계안 제시 및 동의 구하기

### 개략적 설계안

- 메세지의 효과적 전송 가능하도록 설계
- 활성상태인 근망 모든 친구와 항구적 통신상태를 유지하면 됨
    - P2P(peer-to-peer)로 해결 가능?
      → 모바일 단말은 연결상태나 제한적 전력 등의 이유로 공용 백엔드 서버를 사용하는 것이 실용적
    - 백엔드는
        - 모든 활성 사용자의 위치 변화 내역을 수신
        - 수신할 때 마다 거리가 5mile 이내의 모든 활성상태의 친구 단말로 변경 내역을 전달
- QPS 334,000번의 위치 정보 갱신을 처리하기에 마땅하지 않은 설계
    - 사용자 한 명당 400명의 친구를 가진다고 하면 334000 * 400 * 10% = 1400만건의 위치정보 갱신을 처리해야 하고
    - 이 갱신 내역을 사용자 단말로 보내야 함

### 설계안

- 로드밸런서
    - RESTful api 서버 및 양방향 유상태(stateful) 웹 소켓 서버 앞단에 위치, 트래픽 부하 분산
- RESTful API 서버
    - 무상태 api 클러스터. 통상적인 요청/응답 traffic을 처리
    - 그림 2.5
- 웹소켓서버
    - 친구 위치정보 변경을 실시간에 가깝게 처리하는 유상태 서버 클러스터
    - 각 클라이언트는 그 가운데 한 대와 웹소켓 연결을 지속적으로 유지하고 검색 반경 내 친구 위치가 변경되면 해당 내역은 이 연결을 통해 클라이언트로 전송
    - 주변 친구 기능을 이용하는 클라이언트 초기화도 담당.
        - 클라이언트가 시작되면 온라인 상태인 모든 주변 친구의 위치를 해당 클라이언트로 전송하는 역할

### 래디스 위치 정보 캐시

- 활성 상태 사용자의 가장 최근 위치 정보를 캐시하는데 사용
- TTL(time-to-live) 이 지난 사용자는 비활성 상태로 변경하고 캐시에서도 삭제

### 사용자 데이터베이스

- 사용자 데이터, 사용자의 친구 관계 정보를 저장.
- RDBMS나 NoSQL 가능

### 위치 이동 이력 데이터베이스

- 사용자의 위치 변동 이력 보관

### 래디스 pub/sub 서버

- 그림 2.6
- 초경량 message bus
- 새로운 채널을 생성하는건 아주 값싼 연산이고
- 기가바이트급 메모리를 갖춘 최신 레디스 서버에는 수백만 개의 채널(토픽)을 생성할 수 있다
- 웹 소캣 서버를 통해 수신한 특정 사용자의 위치 정보 변경 이벤트는 해당 사용자에게 배정된 펍/섭 채널에 발행
    - 사용자의 친구 각각과 연결된 웹소켓 연결 핸들러는 해당 채널의 구독자로 설정
    - 위치가 바뀌면 웹소켓 핸들러가 호출되고 수신 사용자가 활성상태이면 거리를 다시 계산
    - 갱신된 위치와 갱신 시간을 웹소켓 연결을 통해 해당 친구의 클라이언트 앱으로 전송

### 주기적 위치 갱신  (그림 2.7)

1. 모바일 클라이언트가 위치 변경된 사실을 lb에 전달 → 웹소켓 서버에 전달
2. 웹소켓 서버는 이벤트를 위치 이동 이력 데이터베이스에 저장
3. 웹소켓 서버는 새 위치와 TTL를 위치 정보 캐시에 보관하고
4. 거리 계산에 이용될 웹소켓 연결 핸들러 안의 변수에 해당 위치를 반영
5. 위 과정을 병렬로 수행
6. redis pub/sub 채널에 발행된 위치변경 이벤트가 구독자(웹소켓 이벤트 핸들러, 이벤트 발행자의 온라인 친구)에게 broadcast.
7. 메세지를 수신한 웹소켓 서버는 발행자와 구독자 사이의 거리를 새로 계산

### API 설계

- [서버 API] 주기적인 위치 정보 갱신
    - req: client는 위도, 경도, 시각 정보를 전송하고 응답은 없다
- [클라이언트 API] 클라리언트가 갱신된 친구 위치를 수신하는데 사용할 API
    - data: 친구 위치 데이터, 변경된 시각 Timestamp
- [서버 API] 웹소켓 초기화 API
    - request: client는 위도, 경도, 시각정보 전송
    - response: client는 자기 친구들의 위치 데이터 수신
- [클라이언트 API] 새 친구 구독 API
    - request: 웹소켓 서버는 친구 ID 전송
    - response: 가장 최근의 위도, 경도, 시각 정보 전송
- [클라이언트 API] 구독 해지 API
    - request: 웹소켓 서버는 친구 ID 전송
    - response: 없음

### http 요청

- API 서버는 친구 추가/삭제, 사용자 정보 갱신 등의 작업을 처리할 수 있어야 함

### 데이터 모델

- 위치정보 캐시
    - 주변 친구 기능을 켠 활성 상태 친구의 가장 최근 위치를 보관
    - key: 사용자 ID
    - value: 위도, 경도, 시각
- 위치 정보 저장에 데이터베이스를 사용하지 않는 이유
    - ‘주변 친구’ 기능은 사용자의 현재 위치만을 이용하므로 사용자 위치는 하나만 보관하면 충분하고
    - 읽기, 쓰기 연산속도가 아주 빠른 레디스는 이 목적에 아주 적합
    - ‘주변 친구’ 가 활용하는 위치 정보는 영속성을 보장할 필요가 없다
- 위치 이동 이력 데이터베이스
    - 사용자의 위치 정보 변경 이력은 다음의 스키마를 따르는 테이블에 저장


        | user_id | latitude | longitude | timestamp |
        | --- | --- | --- | --- |
    - 막대한 쓰기 연산 부하를 감당할 수 있고 수평적 규모 확장이 가능한 데이터베이스가 필요
        - 카산드라
        - 막대한 정보를 보관하기 위해 샤딩이 필요한데 user_id를 기준으로 삼는 방안이 가장 기본

## 3단계: 상세 설계

### 중요 구성 요소별 규모 확장성

- API 서버
    - 무상태 서버이므로 cpu 사용률이나 부하, i/o 상태에 따라 자동으로 늘리는 방법은 다양
- 웹소켓 서버
    - 규모를 자동으로 늘리는 것은 easy
    - 유상태 서버이므로 기존 서버를 제거할 때는 주의.
        - Load balancer가 노드 상태를 draining(연결 종료 중)으로 변경하면 새로운 웹 소켓 연결을 만들지 않는다. 충분한 시간이 지난 후 삭제
    - 웹소켓 서버에 새로운 버전의 application s/w를 설치할때도 마찬가지로 주의
    - 결국 유상태 클러스터의 규모를 자동으로 확장하려면 좋은 로드밸런서가 있어야 한다.
- 클라이언트 초기화 (웹소켓 연결이 초기화 되면 )
    1. 위치 정보 캐시에 보관된 사용자의 위치를 갱신, 갱신 후 핸들러 내의 변수에 저장
    2. 사용자 데이터베이스를 뒤져 해당 사용자의 모든 친구 정보를 가지고 옴
    3. 위치 정보 캐시에 batch 요청을 보내어 모든 친구의 위치를 한번에 가지고 온다.
    4. 캐시가 돌려준 친구 위치 각각에 대해 거리를 계산후 웹소켓 연결을 통해 클라이언트에 반환
    5. 웹소켓 서버는 각 친구의 redis pub/sub 채널을 구독
    6. 사용자의 현재 위치를 redis pub/sub 서버의 전용 채널을 통해 모든 친구에게 전송
- 사용자 데이터베이스
    - user_id를 기준으로 데이터를 샤딩하면 관계형 데이터베이스라고 해도 수평적 규모확장이 가능.
    - 웹소켓은 데이터베이스를 api로 연결해 데이터를 가지고 오도록 설계
- 위치 정보 캐시
    - 시스템이 가장 붐빌 때 천만명의 사용자가 활성화 상태이며, 위치 정보 보관에 100바이트가 필요하다고 가정하면, 수 GB 이상의 메모리를 갖춘 최신 redis 서버 한대로도 모든 위치 정보 캐시 가능.
    - 하지만 30초 마다 334K의 갱신 연산을 수행하려면 부담 → user_id 기준으로 샤딩
    - 가용성을 높이려면 각 샤드에 보관하는 위치 정보를 standby node에 복제해두면 됨.
- redis pub/sub 서버
    - 위치 변경 내역 메세지의 라우팅 계층으로 활용
    - 비용이 저렴.
- 얼마나 많은 redis pub/sub 서버가 필요한가?
    - 메모리 사용량
        - 서비스를 사용하는 10억 사용자의 10%인 1억개의 채널이 필요
        - 구독자 한명을 추적하기 위해 내부 해시 테이블과 연결 리스트에 20바이트 정도의 포인터를 저장
            - 200GB(1억 * 20바이트 * 100명의 친구 /10^9 )의 메모리 필요
- CPU 사용량
    - 정보 업데이트 양은 1400만건 / 서버 한대당 가능한 구독자의 수를 보수적으로 100,000 이라고 가정 = 140대 필요. 실제로는 훨씬 적을 것
    - redis pub/sub 서버의 병목구간은 메모리가 아니라 CPU 사용량
- 분산 레디스 pub/sub 클러스터
    - 메세지를 발행할 user_id로 샤딩하면 됨
    - service discovery 컴포넌트를 도입하여 hash ring에 key/value 쌍으로 값을 저장
    - 클라이언트로 하여금 값에 명시된 redis pub/sub 서버에서 발행한 변경 내용을 구독하도록
    - 그림 2.9
    - 웹소켓 서버는 해시링을 참조하여 메세지를 발행할 redis pub/sub 서버를 선정
- 레디스 펍/섭 클러스터 규모 확장시 고려사항
    - 펍섭에 전송되는 데이터 자체는 무상태이다.
        - 구독자에게 데이터가 전송되면 바로 삭제되므로
    - 펍섭 서버는 채널에 대한 상태 정보를 보관한다.
        - 각 채널의 구독자 목록
        - 펍섭 서버 교체, 해시링에서 제거하는 경우 해당 채널의 모든 구독자에게 broadcast 해야 함
            - 기존 채널의 구독 관계를 해지하고 새 서버에 마련된 채널을 구독할 수 있도로고
            - 이 관점에서는 유상태 서버이다.
    - 유상태 서버의 규모를 scaling하는 것은 운영 부담과 위험이 크다.
        - 어느정도 여유를 두고 over provisioning 하는 것이 보통이다.
        - 규모를 늘리게 되면
            - 같은 해시링 위의 다른 여러 서버로 채널이 이동하게 되고
            - service discovery가 모든 웹소켓에 해당 링이 갱신되었다고 알리면 엄청난 재구독 요청이 발생할 것이다.
                - 정보가 누락될 가능성이 있다.
        - 클러스터의 크기를 조정하게 되면
            - 새로운 링 크기를 계산.
            - 해당 링의 키에 매달린 값을 새로운 값으로 갱신
            - 새로운 키값으로 구독하라고 알린다.
            - 그림 2.11
- 친구 추가/삭제
    - 친구가 추가되면 호출될 콜백을 앱에 등록해 두고
    - 이 콜백이 호출되면 웹소켓 서버로 새 친구의 펍/섭 채널을 구독하라는 메세지를 보냄
    - 친구 삭제시에도 콜백을 등록해두고 메세지를 보내어 채널 구독을 지움
- 친구가 많은 사용자
    - 친구 수의 상한을 두도록 설계 (페북은 5000명)
    - 펍/섭 구독관계는 클러스터 내 많은 웹소켓 서버에 분산되어 있을 것이므로 핫스팟 문제는 발생하지 않을 것
- 주변의 임의 사용자를 나타낼 수 있도록 하려면
    - 지오 해시에 따라 구축된 펍섭 채널 풀을 두고
    - 지오해시 격자마다 채널을 하나씩 만들어 두고
    - 해당 격자내의 모든 사용자는 그 채널을 구독한다
    - 격자 경계 부근의 사용자를 잘 처리하기 위해 주변 격자의 채널도 구독
- 래디스 펍/섭의 대안
    - 얼랭
        - 고도로 분산된 병렬 애플리케이션을 위해 고안된 프로그래밍 언어이자 런타임 환경
        - 경량 프로세스
            - BEAM VM에서 실핸되는 entity
            - 리눅스 프로세스 생성 비용에 비해 엄청나게 저렴
            -