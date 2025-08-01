## 09. 알림/경보 서비스 설계
- 다른 기능으로의 재사용 가능성

#### 9.1 기능 요구사항
- 가동 시간 모니터링용이 아님 : 중단 경보를 트리거 하지 않기 때문에 독립된 인프라에서 제공
- 사용자와 데이터 : 발신자, 수신자, 관리자
- 수신자 채널 : 브라우저, 이메일, 문자, 전화, 앱...
- 템플릿 : 공통된 내용을 재활용발신 하기 위한 기능, CRUD API
- 트리거 조건 : 자동 / 수동
- 구독자, 발신자 그룹, 수신자 그룹 관리 : CRUD
- 사용자 기능
  - 중복 발송 회피
  - 과거 알림요청 확인
  - 템플릿 관리
  - 알림 상태 조회
  - 알림의 우선순위 조정

#### 9.2 비기능적 요구사항
- 스케일 : 대용량
- 성능 : 몇 초 내에 전달이 돼야 함. 우선순위 부여 필요
- 고가용성 : 99.999% (1년중 약 5분 다운타임 허용)
- 내결함성 : 수신자가 못받는 상황이면 다음기회에 받아야 함
- 보안 : 인증된 사용자만 발송 가능
- 프라이버시 : 수신 거부 가능

#### 9.3 초기 고수준 아키텍처
- 고려사항
  - 알림 요청 사용자는 단일 인터페이스를 기대
  - 내부 채널(발송 방식)에 따라 특화된 로직 제공
  - 작업 생성기 : 공통 채널 서비스 로직을 다른 서비스에서 중앙 집중화 가능
  - 동기 메커니즘에는 확장성이 없음. 이벤트 스트림와 비동기 기술 필요
- 고수준 아키텍처
  - 고객 -> 알림 서비스 -> 외부 알림 서비스
  - 세부 : 프론트엔드 -> 발행자 -> 알림큐 -> 소비자 -> 다양한 채널 알림, 이벤트 큐와 공유 로그,
- 알림 서비스와 채널 서비스와 다른 서비스 간에 연결이 없음
  - 내결함성 높아짐
  - 재사용성, 확장성
  - 세부 동작 방식에 대한 관심사 분리
- 프론트엔드 서비스에서의 역할
  - 속도 제한
  - 프라이버시
  - 보안
  - 모니터링, 분석, 경보
  - 캐싱
- 각 채널에 따른 우선순위 분리 가능(토픽 분리)
- 각 레이어 에서의 로깅 (문제 해결, 감사 등)

#### 9.4 객체 스토리지: 알림 구성과 전송
- 알림에 큰 파일이나 객체가 포함되면 비 효율적
- 메타데이터 서비스를 추가하여 최초 알림 트리거 시점에서 등록 후 재 활용

#### 9.5 알림 템플릿
- 많은 알림 이벤트가 소량의 개인 설정으로 거의 중복됨
- 템플릿의 CRUD가 가능 해야함
- 알림 발송시점에서는 템플릿 ID를 포함
- 추가기능 : 템플릿 제어, 재사용, 검색 등

#### 9.6 예약된 알림
- 에어플로 서비스나 작업 스케줄러 서비스를 사용해서 트리거링 가능
- 다른 알림들과 경쟁될수 있음, 로그로 확인하고 확장이 필요
> 예약 알림과 신규 알림간의 우선순위 또는 프로세스가 정확히 다르게 처리되야 하지 않을까?

#### 9.7 알림 수신자 그룹
- 목적지/주소를 관리하는 수신자 그룹
- 이를 관리하는 API
> 주소 그룹의 사용자 변경은 어떻게 처리 해야 맞을까? 경험 : 발송 시점 기준으로 무시

#### 9.8 구독 취소 요청
- 알림 종류별 수신여부 설정
- 구독취소 : 클라이언트 + 서버 사이드 모두 구현 (버그 방지 등)

#### 9.9 실패한 전달 처리
- 수신자의 기기에 도달하지 못하는 경우
- 차단 또는 버그
> 발신 성공 여부를 다 체크하기엔 서버가 너무 부담이지 않을까
> 알림 목록 (새소식) 으로 처리하는게 좋지 않을까
> "알림" 은 말 그대로 시급도가 높은 UX

#### 9.10 중복 알림에 관한 클라이언트 사이드 고려사항
- 서버/클라이언트 모두 중복 제거 처리 
- 클라이언트는 로컬 스토리지 활용

#### 9.11 우선순위
- 우선순위별 토픽 다르게 처리
- 가중치 방식으로 동적 토픽 선택 가능
> 우선순위 높은 토픽의 LAG가 많은 상태에서 처리되는 것과
> 우선순위 낮은 토픽의 LAG가 0인 상태에서 처리되는것. 후자가 더 빠르지 않나?

#### 9.12 검색
- 알림 설정 검색
- 템플릿이나 주소 그룹 검색

#### 9.13 모니터링과 경보
- 알림상태 추적
- 알림 발송 시점에서의 시스템 상태 로그
- 이상감지 : 발송 수 vs 수신 수
> 특정 사용자의 특정 알림 모니터링 현황 - VOC

#### 9.14 알림/경보 서비스의 가용성 모니터링과 경보
- 알림/경보 서비스를 모니터링 하는 별도의 클라이언트 데몬(하트비트)
> 어디까지 모니터링 할것인가? (A <- B <- C <- D <- ...)

#### 9.15 기타 논의 가능한 주제
- 알림 발송 양에 따른 스케일 조정...?
  > 리밸런싱 발생하지 않나...
- 이미 발송한 알림 수정
  > 발송한 알림 제거(클라이언트) 및 재발송
- 채널별 발신 속도 제한
  > BTS sleep...
- 분석 : 각 데이터 수집
- 확장을 어떻게 할 것인가