## 10. 데이터베이스 배치 감사 서비스 설계

- 데이터 품질 (Data quality)
  - 정확성
  - 완전성
  - 일관성
  - 유효성
  - 고유성
  - 적시성

#### 10.1 감사는 왜 필요한가?
- 반대 의견이 있지만
  - 애초에 올바른 데이터만 인입하는걸 보장한다거나
  - 외부 감사가 아닌 내부 어플리케이션에 의해 처리 해야 한다거나
- 그럼에도
  - 버그로 감지되지 못한 데이터
  - 지난 데이터
  - 비즈니스의 변경
  - 감사는 또다른 유효성 검사 계층
  - 중복 혹은 누락

#### 10.2 SQL 쿼리 결과에 대한 조건문으로 유효성 검사 정의
- 최근 하루 데이터를 조회한 후 + 그 데이터의 특정 값이 비즈니스 기준과 다른지 점검
  - 만료 쿠폰일자 조회, 총 판매수 제한, 특정 지역 조회, 통계 데이터 비율 점검

#### 10.3 간단한 SQL 배치 감사 서비스
- 감사 스크립트를 만들고 (위에서 했던 SQL)
- 파라미터에 의해 기준값을 주입할수 있도록 템플릿화 한 다음
- 실패가 발생했을때 경보를 트리거 할 수 있는 로직을 추가

#### 10.4 요구사항
- 주기적으로 실행될수 있도록 : 주기
- 실패시 경보 발생
- 과거로그 조회 가능(실패, 수행 시간 등)
- 특정 시간 이내에 수행 완료
  > 쿼리가 15분..?

- 비기능적 요구사항
  - 스케일
  - 가용성
  - 보안
  - 정확성

#### 10.5 고수준 아키텍처
- 책임/관심사 분리 : 유지보수(관리포인트)
- 쿼리 실행 -> 결과를 조건문으로 실행
- 사용자 -> 백엔드 <-> ETL
- 백엔드 -> 로깅 서비스 -> 모니터링 서비스 -> 경보 서비스

#### 10.6 데이터베이스 쿼리 제약
- 쿼리 실행 시간 제한
- 제출 전 쿼리 문자열 확인 (UI)
- 기능 사용에 대한 가이드 문서

#### 10.7 과도한 동시 쿼리 방지
- 동시 실행되는 쿼리 수를 제한
- "쿼리 서비스" : 백엔드 -> 다양한 쿼리 실행 서비스

#### 10.8 데이터베이스 스키마 메타데이터의 사용자
- 사용자의 쿼리 작성을 돕기 위해 서비스는 스키마 메타데이터에서 작업 구성을 자동으로 도출 가능

#### 10.9 데이터 파이프라인 감사
- DAG(Directed Acyclic Graph) 형태
- 상위 작업이 실패하면 하위 감사를 비활성화
- 비활성화된 작업의 소유자와 그 하위의 소유자에게 경보 트리거

#### 10.10 로깅, 모니터링, 경보
- 로깅/모니터링 후 경보
  - 상태와 상세 시간
  - 실패/오류 이유
  - 비정상 응답
  - 높은 리소스 사용

#### 10.11 기타
- IDC간 데이터 일관성 감사
- 업/다운 스트림 데이터 비교
- 확장성 고려
- 동일한 작업에 대한 관리(중복 제거?)