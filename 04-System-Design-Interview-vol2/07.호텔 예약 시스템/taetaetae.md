# 7장 호텔 예약 시스템
- 1단계 : 문제 이해 및 설계 범위 확정
  - 개략적 규모 추정
    - 총 5천개, 100만개의 객실
    - 평균 객실의 70%가 사용중, 투숙기간 평균 3일
    - 일일 예약 건수 : 약 24만
    - 초당 예약 건수 : 약 3 TPS
  - 비기능 요구사항
    - 높은 수준의 동시성 : 성수기, 대규모 이벤트 기간에 고객이 몰릴 가능성
    - 적절한 지연 시간 : 예약요청 처리에 몇초 정도 걸리는건 괜찮음
  - 페이지별 흐름
  	- 호텔/객실 상세 페이지 : 조회
  	- 예약 상세 정보 페이지 : 조회
  	- 객실 예약 페이지 : 트랜잭션 발생
- 2단계 : 개략적 설계안 제시 및 동의 구하기
  - API 설계
    - 호텔/객실/예약 CRUD
    - 각 API 별 권한 분리
    - 신규 예약시 이중 예약 방지를 위한 멱등키 필요
  - 데이터 모델
  - 개략적 설계안
    - CDN : 정적 콘텐츠 캐싱
    - 공개 API 게이트웨이 : 처리율 제한, 인증, 엔드포인트 기반의 라우팅
    - 내부 API : 사설망 기술을 활용하여 외부와의 차단
    - 호텔 서비스 : 상세 정보, 캐시 가능
    - 요금 서비스 : 미래에 어떤 요금을 받을지에 대한 데이터 제공, 손님이 몰리느냐에 따라 요금이 달라짐
    - 예약 서비스 : 객실의 상태 갱신 역할도 담당
    - 결제 서비스 : 결제 상태에 따라 상태 업데이트
    - 호텔 관리 서비스 : 어드민
  - MSA의 구성, 필요에 따라 RPC 프레임워크를 사용하기도 함
- 3단계 : 상세 설계
  - 개선된 데이터 모델
    - 예약시 특정 객실이 아니라 특정 객실 유형을 예약하는 경우
    - 특정 날짜에 예약 가능한 정보들이 담긴다.
    - 용량문제라면 샤딩을 하거나 냉동 저장소를 활용
  - 동시성 문제
    - 사용자가 두번 예약
      - 클라이언트 측에서 처리 가능, 우회 쉬움
      - 멱등키로 해결
    - 하나의 방을 두명의 사용자가 동시 예약
      - 비관전 락(트랜잭션 대기)
      - 낙관적 락(버저닝)
      - 데이터베이스 제약 조건(constraint)
  - 시스템 규모 확장
    - 데이터 베이스 샤딩
    - 캐시 : TTL, 서비스간 분리, 갱신 문제
  - 마이크로 서비스 아키텍처에서의 데이터 일관성 문제에 대한 해결 방안
    - 2단계 커밋
    - 사가
- 4단계 : 마무리
  - 데이터베이스의 변화
  - 경쟁조건에서의 해결방안
  - 규모 확장 및 MSA에서의 데이터 일관성 처리
