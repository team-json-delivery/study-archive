# 1장. 쿼리 응답 시간
- 성능은 곧 쿼리 응답 시간이다.
- 쿼리 응답시간 == 응답시간, 쿼리시간, 실행시간, 쿼리 지연시간
  - laytency : 각각의 시스템 혹은 장비에 내재된 자체 지연 시간
  - delay : 각각의 시스템 지연 시간이 모아진 지연 시간
- 같은 쿼리가 백만 개의 행을 조회할 수는 있지만, 조회된 백만 개의 행을 우리가 경험하지는 않는다
  > 우리에게는 시간이 의미가 있다.
- 결국 MySQL 성능에서 핵심 지표인 쿼리 응답 시간 향상에 집중해야 한다.
- 쿼리 메트릭
  - 소스
    - 슬로 쿼리 로그
      - 기본으로 비활성화되어 있지만 MySQL을 다시 시작할 필요 없이 바로 활성화할 수 있다.
    - 성능 스키마
      - MySQL을 다시 시작해야 함
  - 집계
    - 정규화
    - SQL 문을 정규화 하여 다이제스트 텍스트를 생성한 다음 이에 대해 SHA-256 해시를 계산하여 다이제스트 해시를 생성 (고유함)
    - 쿼리 추상화는 고유하진 않지만 간결해서 유용하다.
  - 보고
    - 쿼리 프로파일
      - 느린 쿼리가 표시됨
      - 일반적인 집계치
        - 쿼리 총 시간 : 실행시간(쿼리당)의 총 합
        - 실행 시간 비율 : 쿼리 총 시간(쿼리당)을 실행 총 시간(모든 쿼리)으로 나눈 값
        - 쿼리 부하 : 쿼리 총 시간(쿼리당)을 클럭 타임으로 나눈 것, 동시에 실행되는 쿼리의 다중 인스턴스와 같은 동시성을 미묘하게 나타냄
    - 쿼리 보고서
      - 하나의 쿼리에 대해 알아야 할 모든 것을 보여줌
      - 쿼리 샘플, EXPLAIN 계획, 테이블 구조 등의 메타데이터가 포함
- 쿼리 분석
  - 목표는 느린 응답 시간을 해결하는 것이 아니라 '쿼리 실행'을 이해하려는 것.
  - 느린 응답 시간을 해결하는 행위는 쿼리 분석 후 쿼리 최적화 과정에서 이루어짐
  - 쿼리 메트릭
    - 쿼리시간 : 잠금시간이 포함되어 있음
    - 성능 스키마의 잠금시간에는 로우락 대기가 포함되지 않는다.
    - MySQL에서 성능 스키마로 이벤트를 수집하며, 이벤트는 다음과 같은 계층 구조로 구성된다.
      트랜잭션: 최상위 이벤트
      명령문: 쿼리 메트릭이 적용되는 쿼리
      단계: 명령문 실행 과정 내의 단계. 명령문 구문 분석, 테이블 열기, 파일 정렬 수행과 같은 과정을 포함
      대기: 시간이 걸리는 이벤트
    - 잠금시간
      - 쿼리를 실행하는 동안 잠금을 획득하여 사용하는 시간
      - 락 종류
        공유 락(shared lock): 다른 공유 락이 동시에 획득되지만, 배타적 락은 얻지 못한다.
        배타적 락(exclusive lock): 다른 공유 락과 배타적 락을 얻지 못한다.
      - 성능 스키마의 잠금 시간: 로우 락 대기가 포함되지 않고 테이블과 메타데이터 락 대기만 포함한다.
      - 슬로 쿼리 로그의 잠금 시간: 로우 락, 테이블 락, 메타데이터 락 모두 포함한다.
      - 읽기에는 비잠금 읽기(nonlocking reads)와 잠금 읽기(locking reads)가 있다.
        - 잠금 읽기
          - SELECT ... FOR UPDATE : SELECT한 행에 대해서 배타적 로우 락이 잡힌다.
          - SELECT ... FOR SHARE : SELECT한 행에 대해서 공유 로우 락이 잡힌다.
        - 비잠금 읽기
          - 일반적인 SELECT
          - 메타데이터 락과 테이블 락을 획득해야 된다.
    - 조회된 행
      - MySQL이 쿼리 조건 절에 일치하는 행을 찾으려고 접근한 행의 수 : 쿼리와 인덱스의 선택도를 나타낸다.
      - 선택도가 높을 수록 MySQL이 일치하는 행을 조회하는 데 낭비하는 시간이 줄어든다.
    - 보낸 행(rows sent)
      - 클라이언트에 반환된 행의 수를 나타낸다.
      - 보낸 행 = 조회된 행
        - 이상적인 경우. 특히 전체 행의 백분율로 계산했을 때 상대적으로 값이 작고, 허용할 수 있는 쿼리 응답 시간일 때가 이상적이다.
        - 비율과 관계없이 보낸 행과 조회된 행이 같고 값이 의심스러울 정도로 높으면 쿼리가 테이블 스캔을 유발한다는 것을 의미으로 성능 면에서 매우 안 좋은 상황임을 암시한다.
      - 보낸 행 < 조회된 행
        - 쿼리나 인덱스의 선택도가 좋지 않다는 신호다.
        - 이 쿼리로 인해 MySQL은 많은 시간을 낭비하고 있다는 뜻이다.
      - 보낸 행 > 조회된 행
        - 드문 경우다.
        - MySQL이 쿼리를 최적화 할 수 있을 떄와 같은 특별한 조건에서 발생한다.
        - 예) SELECT COUNT(id) FROM t2는 COUNT(id) 값에 대해 1개 행을 보내지만 0개 행을 조회한다.
    - 영향받는 행(rows affected)
      - 삽입, 갱신, 삭제된 행의 수를 나타낸다.
      - 해당하지 않는 행이 변경되면 심각한 버그가 발생하므로 주의해야된다.
    - 셀렉트 스캔(select scan)
      - 쿼리가 2개 이상의 테이블에 접근할 때 첫 번째로 접근한 테이블에서 수행한 전체 테이블 스캔 횟수를 나타낸다.
      - 쿼리가 인덱스를 사용하지 않는다는 것을 의미하므로 일반적으로 성능에 좋지 않다.
    - 셀렉트 풀 조인(select full join)
      - 조인된 테이블을 대상으로 전체 테이블을 스캔한 수를 나타낸다.
      - 셀렉트 스캔과 유사하지만 더 나쁘다.
      - 셀렉트 풀 조인은 조인 수가 이전 테이블 행의 곱과 같아서 셀렉트 스캔보다 더 나쁘다.
    - 디스크에 생성된 임시 테이블
      - 쿼리가 메모리에 임시 테이블을 만드는 것은 정상이다.
      - 그러나 메모리에 임시 테이블이 MySQL이 임시 테이블을 디스크에 쓴다. 이는 응답 시간에 영향을 미친다.
    - 쿼리 카운트
      - 쿼리 실행 횟수를 나타낸다.
      - 매우 낮고 쿼리가 느리지 않는 한 기준이 없고 임의적이다.
  - 메타데이터와 애플리케이션
    - 메타데이터로는 EXPLAIN 계획과 테이블 구조 등이 있다.
    - EXPLAIN과 SHOW CREATE TABLE은 각각 EXPLAIN 계획과 테이블 구조를 보여준다.
    - 애플리케이션이 쿼리를 실행하는 이유가 무엇인지 분석하고, 쿼리를 간결하게 하거나 제거하는 것도 중요하다.
  - 평균 : 쿼리 수가 매우 크거나 작은 몇 개의 값이 평균 음답 시간을 왜곡할 수 있다.
  - 백분위
    - 평균이 갖는 문제를 보완한다.
    - P95: 샘플의 95%가 이보다 작거나 같은 값이다.
      예시) P95가 100ms와 같으면 값의 95%가 100ms보다 작거나 같은 값으며, 5%가 100ms보다 크다.
    - 평균보다 대표성을 띈다.
  - 최대
    - 백분위수가 갖는 문제를 보완한다.
    - 세계 어딘가에서 일부 애플리케이션 사용자는 최대 쿼리 응답 시간을 경험하였거나 몇 초 후에 절망하고 떠난다.
- 쿼리 응답 시간 개선
  - 직접 쿼리 최적화
    - 쿼리와 인덱스를 변경하는 것
    - 쿼리 분석, EXPLAIN, 인덱스 등으로 쿼리 최적화를 할 수 있다.
    - MySQL 매뉴얼에 명시된 "SELECT 문 최적화"
      범위 최적화
      인덱스 머지 최적화
      해시 조인 최적화
      인덱스 컨디션 푸시다운 최적화
      다중 범위 읽기 최적화
      Constant-Folding 최적화
      IS NULL 최적화
      ORDER BY 최적화
      GROUP BY 최적화
      DISTINCT 최적화
      LIMIT 쿼리 최적화
  - 간접 쿼리 최적화
    - 데이터와 접근 패턴을 변경하는 것
    - TRUNCATE TABLE을 통해 데이터가 없으면 거의 0에 가까운 시간에 모든 쿼리를 실행할 수 있다.
    - 직접 쿼리 최적화로 문제가 해결되면 간접 쿼리 최적화는 하지 않는 게 바람직하다.
    - 간접 쿼리 최적화는 더 많은 노력이 들어가기 때문이다.
- 언제 쿼리를 최적화해야 할까?
  - 느린 쿼리를 수정하려고 시간을 투자하는 게 항상 효율적인 건 아니기에 매번 쿼리를 최적화해서는 안된다.
  - 쿼리를 최적화해야 하는 상황 3가지
    - 성능이 고객에게 영향을 미칠 때
    - 코드 변경 전후
    - 한 달에 한 번: 코드와 쿼리가 변경되지 않더라도 데이터와 접근 패턴이라는 적어도 2가지 요소가 변경된다.
- MySQL을 더 빠르게
  - MySQL이 동일한 시간 내에 더 많은 작업을 수행하도록 하는 3가지 옵션
    - 시간의 본질 바꾸기
    - 응답 시간 단축: 직접 쿼리 최적화. 간접 쿼리 최적화.
    - 부하량 증가: 더 많은 CPU 코어를 사용하여 응답
  - 하드웨어 성능이 한계에 도달하면 불안정해지므로, MySQL의 속도를 높이려면 직접 및 간접 쿼리 최적화라라는 여정을 시작해야만 한다.