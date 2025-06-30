# Chapter 4 : Scaling Databases

---

## 1. 스토리지 서비스 개요

### Stateless vs. Stateful
- Stateless 서비스는 확장과 장애 대응이 쉽다.
- Stateful 서비스는 상태를 유지하며 복잡성 증가. Paxos, Quorum 등으로 일관성 보장 필요.
- 상태는 가능한 한 외부 저장소에 위임하여 stateless 구조 유지.

### 저장소 유형 분류
- **Relational DB (SQL)**: 테이블 기반, ACID 보장.
- **NoSQL**:
  - **Key-Value**: Redis, Memcached (빠른 조회, 캐시 용도).
  - **Column-Oriented**: Cassandra, HBase (대용량 필터링에 적합).
  - **Document**: MongoDB (JSON, YAML 등 문서 저장).
  - **Graph**: Neo4j, RedisGraph (관계형 데이터 표현).
- **기타 저장소**:
  - **File storage**: 디렉토리/파일 기반.
  - **Block storage**: 저수준 블록 단위 저장.
  - **Object storage**: S3 등 HTTP 기반, 수정 불가, 정적 파일 저장에 적합.

---

## 2. 저장소 선택 시 고려사항

- 파일 vs. DB 저장은 데이터 크기와 처리 방식에 따라 결정.
- 256KB 미만 → DB / 1MB 초과 → 파일 시스템 / 중간 영역은 쓰기 빈도, 덮어쓰기 여부 고려.
- DB 저장 시 큰 객체(BLOB)는 복제 지연, 메모리 효율 저하 가능.

---

## 3. 데이터베이스 확장 기술

### Replication (복제)
- **단일 노드 한계 극복**: 저장 용량, 처리량, 지연시간, 장애 복구 등을 위해 복제 필요.
- **읽기 확장**: 다수 복제본 활용.
- **쓰기 확장**: 복잡함. 아래 전략 필요.

### 복제 전략
- **Single-leader replication**
  - 모든 쓰기는 하나의 리더에서 수행. 팔로워에게 비동기 복제.
  - 리더 장애 시 secondary leader로 승격.
  - 읽기 확장에 적합하나 쓰기 확장에는 한계.

- **Multi-leader replication**
  - 여러 리더가 독립적으로 쓰기 가능. 각 리더는 다른 리더로 복제.
  - **Race condition** 발생 가능 (예: 동시 DELETE/UPDATE).
  - **Clock skew**로 인해 순서 결정 어려움 → 일관성 처리 필요.

- **Leaderless replication**
  - 모든 노드가 동일 권한. 쓰기/읽기 Quorum으로 일관성 조정.
  - 예: Cassandra, Dynamo
  - UPDATE/DELETE는 최종 일관성 수준. - eventual consistency

- **HDFS replication**
  - NameNode가 메타데이터 담당, DataNode가 블록 저장.
  - append-only. UPDATE/DELETE 미지원 (tombstone 사용).
  - Hive 등과 연계해 대용량 분석 처리.

---

## 4. Sharding과 저장소 확장

- 단일 노드 용량 초과 시 샤딩 필요.
- **샤딩된 RDBMS**: Amazon RDS, MySQL 샤딩
  - JOIN, Aggregation 등 SQL 기능 제약.
  - 샤딩 키 설계 중요.

---

## 5 Aggregating Events: 이벤트 집계로 데이터베이스 쓰기 줄이기

---

### 1. 왜 이벤트를 집계해야 하는가?

### 데이터베이스 쓰기는 비용이 높고 확장하기 어렵다
- 읽기(SELECT)는 복제와 캐싱으로 수월하게 확장 가능.
- 쓰기(INSERT/UPDATE/DELETE)는 **병목의 주범**:
  - 동시성 충돌, 인덱스 갱신, 디스크 I/O, ACID 보장 비용.
- 따라서, **쓰기 빈도 자체를 줄이는 방법**이 중요.

### 이벤트 집계(Aggregation)의 핵심 목표
- **여러 이벤트를 하나의 레코드로 압축하여 기록**.
- DB 크기 증가 속도도 감소.
- 특히 **정확한 timestamp가 필요 없는 경우에 적합**.
- 스트리밍 파이프라인과 결합 시 실시간 처리도 가능.

---

### 2. 이벤트 집계 기법 개요

### Sampling (샘플링)
- 모든 이벤트를 기록하지 않고, **일부만 선택적으로 저장**.
- 예: 매 100개 중 1개, 랜덤 선택, 조건 기반 필터링.

### Aggregation (집계)
- 유사 이벤트들을 그룹핑하여 하나로 합침.
- 예: `count`, `sum`, `average`, `min`, `max` 등 통계 집계.
- 이벤트의 발생 시점이 중요하지 않을 때 사용.

---

### 3. 단일 계층 집계 (Single-tier Aggregation)

### 아키텍처
- **로드 밸런서**가 이벤트를 여러 집계 호스트로 분산.
- 각 호스트는 **메모리 내 해시 테이블**을 유지:
  - 예: `{A: 3, B: 5, C: 2}`
- 일정 주기(예: 5분) 또는 메모리 임계치 초과 시 DB에 flush.

### 장점
- 단순한 구조.
- write 빈도 대폭 감소 → DB 부하 완화.
- 메모리 캐시 기반 → 처리 속도 빠름.

### 단점
- 단일 계층은 대규모 트래픽에 한계.
- 장애 발생 시 **복구 어려움** → replication 필수.

---

### 4. 다계층 집계 (Multi-tier Aggregation)

### 구조
- 이벤트 → 로드 밸런서 → Tier 1 호스트 → Tier 2 → Tier N → DB
- 각 계층은 이전 계층의 결과를 받아 **추가 집계 수행**.
- **최종 계층만 DB에 기록**.

### 이점
- 확장성 극대화 (수천 개 노드까지 확장 가능).
- 계층마다 처리량 분산 가능.
- 불필요한 DB writes를 더욱 줄임.

### 단점
- **지연(latency)** 증가: 각 계층마다 시간 필요.
- **복잡도** 증가: 로깅, 모니터링, 장애 복구 설계 필요.
- **일관성 지연** 발생 가능 → 이벤트가 즉시 반영되지 않음.

---

### 5. 파티셔닝을 통한 트래픽 분산

### 레벨 7 로드 밸런서를 통한 분산
- **이벤트의 key 값**을 기준으로 파티션을 정의.
- 예: A–I, J–R, S–Z로 나누고 각 파티션에 할당된 호스트 분리.
- **hot partition 방지**:
  - 특정 key 범위에 트래픽 몰릴 경우, 호스트 수 조정.
  - 예: A–I (3개), J–R (1개), S–Z (2개).

### 파티션 다이내믹 조정
- 트래픽 분포 따라 파티션 폭/개수 유연하게 조정.
- 예: {A-B, D-F}, {C, G-J}, {K-S}, {T-Z} 등 **불균등 파티션** 설계 가능.

---

### 6. 대규모 Key-space 처리

### 문제
- 수백만 개의 고유 key가 있을 경우, 메모리 한계 초과 가능.

### 해결 전략
- **상위 계층 호스트의 key-space 제한**:
  - 각 호스트가 최대 메모리의 절반 이하만 사용하도록 제한.
- **더 낮은 계층 호스트에 더 많은 메모리 할당**:
  - 예: 1계층(2호스트) → 2계층(1호스트)

---

### 7. 복제 및 장애 허용 (Replication & Fault Tolerance)

### 문제점
- 중간 계층 호스트 장애 시:
  - 해당 계층의 **모든 집계 데이터 손실**.
  - **상위 계층의 오버플로우 발생** 가능.

### 해결 방법
- **Checkpoints + Dead-letter queue**:
  - 주기적 저장 + 실패 이벤트 재처리 큐.
- **각 집계 노드를 독립 서비스화**:
  - 예: Redis-backed 캐시 + 다수의 무상태 worker pods 구성.
  - 예시 구성:
    - Load balancer → Service (3 pods) → Redis (shared) → DB

### 추가 조치
- **flush 성공 시 데이터 삭제** → Redis 크기 관리.
- **Terraform + Kubernetes** 조합으로 인프라 정의.

---

### 8. 집계 단위 설계

### Aggregation Unit
- 집계 계층을 **"aggregation unit"** 이라는 독립 서비스로 구성.
- 각 단위:
  - 복수의 stateless pods (예: 3개).
  - Redis 등의 메모리 저장소를 공유.
  - 플러시 이후 데이터를 삭제 → 메모리 절약.

---

### 9. 이벤트 전처리와 추후 보장

### Saga, Quorum write 등 활용
- **단일 이벤트라도 안정적으로 복제**되도록 보장 필요.
- 장애 시 rollback, dead-letter 처리 포함.


---

## 6. Batch & Streaming ETL

### 배치 처리 (Batch)
- cron + Python + SQL 조합으로 DAG 구성 가능.
- 예: Airflow, Luigi → 확장성, UI 제공.

### 스트리밍 처리 (Streaming)
- Kafka, Flink 등 사용.
- **Pull vs. Push** 전략 고려:
  - Pull은 컨슈머가 처리량 제어 가능.
  - Push는 빠른 실시간 처리에 적합 (단, 복잡도↑).

- **Lambda Architecture**:
  - 배치 + 스트리밍 병행. 빠른 반응성과 정확성 조화.
  - 대안: Kappa Architecture (stream-only)

---

## 7. Denormalization

- Normalization은 데이터 중복 방지, 일관성 ↑
- 하지만 JOIN 성능 저하로 인해 실무에서는 Denormalization 사용 많음.
- SELECT 성능 향상, 간결한 쿼리 작성 장점.

---

## 8. Caching 전략

### 장점
- 응답 시간 단축 (성능 ↑)
- 트래픽 감소 (스케일링 ↑)
- 장애 시 일부 응답 가능 (가용성 ↑)

### 캐싱 계층
- Client → API Gateway → Backend → DB
- CDN 활용 가능

### 캐시 패턴
- **Cache-aside (Lazy load)**: 앱에서 DB 조회 후 캐시에 저장
- **Read-through**: 캐시가 DB 접근, 앱은 캐시만 접근
- **Write-through**: 모든 쓰기를 캐시 → DB
- **Write-back**: 캐시에서 쓰고, 일정 주기로 DB에 flush
- **Write-around**: DB에 직접 쓰고, 캐시는 읽기 때만 갱신

---

## 9. Cache Invalidation & Warming

### Invalidation 전략
- TTL (max-age)
- Fingerprinting (버전으로 파일명 변경)
- Replacement policy (LRU, FIFO 등)

### Cache Warming
- 미리 캐시를 채워 사용자 첫 요청을 빠르게 응답
- 단점: 복잡도, 비용 증가 / 정확한 예측 어려움

---

## 요약

- 저장소 선택, 복제 전략, 샤딩, ETL, 캐싱까지 모든 데이터 관련 설계는 **확장성**, **일관성**, **성능** 사이의 트레이드오프를 동반함.
- 시스템 설계 면접에서 각 기술의 장단점, 적용 시점, 제약 조건 등을 명확히 이해하고 설명할 수 있어야 함.

