# Non-Functional Requirements 요약

## 1. 개요

시스템 설계에서 기능적 요구사항뿐 아니라 **비기능적 요구사항**을 반드시 함께 논의해야 한다. 고객이 이를 명확히 정의하지 않는 경우가 많으며, 면접에서도 이 부분을 먼저 명확히 하고 시작해야 한다.

### 주요 비기능 요구사항
- Scalability (확장성) : 크기를 `쉽게` 키울 수 있는가
- Availability (가용성) : 시스템이 안 죽고 잘 살아 있는가
- Performance / Latency / Throughput (성능, 지연시간, 처리량) : 얼마나 빠르게 응답 하는가
- Fault-tolerance (장애 허용) : 시스템 전체가 의존적이지 않은가
- Consistency (일관성) : 경우에 따라 다른 응답을 주진 않는가
- Accuracy (정확성) : 데이터가 정확한가?
- Security (보안) : OAuth
- Privacy (개인정보 보호) : PII가 잘 격리되어 있는가
- Complexity / Maintainability (복잡성, 유지보수성) : 유지보수가 어렵지 않은가
- Cost (비용) : 비용 효율적인 시스템인가

---

## 2. Scalability (확장성)

### 수직 확장 (Vertical Scaling)
- 장비 스펙 향상 (CPU, RAM, SSD 등)
- 단점: 비용 증가, 물리적 한계, 다운타임 필요

### 수평 확장 (Horizontal Scaling)
- 여러 호스트로 트래픽 분산
- Stateless 서비스와 Load Balancer 사용
- 무상태 서비스가 수평 확장에 유리

### Load Balancer
- Level 4: TCP 기반
- Level 7: HTTP 기반, 인증, TLS 종료 등 가능
- Sticky Session, Session Replication 고려

---

## 3. Availability (가용성)

### 정의
- 시스템이 정상적으로 요청을 처리할 수 있는 시간의 비율
- 예: 99.99% = 월 약 4.38분 다운타임 허용
- S3의 고가용성이 내구성 99.999999999% - 1조개당 1개의 object가 사라질 가능성이 있음, 99.99%의 가용성으로 알려져 있음

### 고가용성 전략
- Multi-region replication
- 비동기 처리 (이벤트 소싱, 사가)
- 즉시 응답이 아닌 `ack` + 나중 처리 가능 여부 확인
- 서비스 성격에 따라 필요한 수준 결정

### Fault-tolerance (장애 허용)
- 일부 구성 요소가 실패해도 시스템이 계속 동작할 수 있는 능력 : 의존성 낮음
- 내결함성은 metric으로 집계되는 것은 아니고 시스템의 특성 : 원할한 오류 처리를 위한 실패에 대한 설계에 기반한다
  - 외부 api의 실패는 제어할 수 없으므로
  - 복제/중복, 서킷 브레이커, 지수 백오프와 재시도(DLQ), 응답 캐싱 등을 통해 내결함성을 높일 수 있다
  - 폴백 패턴은 업스트림의 실패시 다른 경로로 대체하는 것(결제 서비스가 여러개라면 한개를 제외할 수도 있을듯)
### 주요 기법
- Replication: 리더-팔로워, 다중 데이터센터
- Circuit Breaker: 실패 요청 차단
- Retry with Exponential Backoff + Jitter
- Dead Letter Queue
- Caching for graceful degradation
- Checkpointing (Kafka, Flink 등)
- Bulkhead Pattern: 풀 분리로 격리
- Fallback Pattern: 캐시/타사 서비스 활용

---

## 4. Performance / Latency / Throughput

- Latency: 사용자 요청 응답 시간
- Throughput: 초당 처리량

### 성능 향상 기법
- 지리적 위치 최적화 (데이터센터 위치)
- CDN, 캐시
- 경량화 프로토콜 (RPC, Protobuf 등)
- REST → RPC 전환
- 배치 처리, 스트리밍 처리

---

## 5. Consistency (일관성)

### CAP vs ACID
- CAP Consistency: Linearizability
- ACID Consistency: 관계형 제약 유지

### 기술적 접근
- Full Mesh: 단순하지만 확장성 낮음
- Coordination Service: 리더 선출 (e.g., ZooKeeper, Raft)
- Distributed Cache (e.g., Redis)
- Gossip Protocol: 무작위 전파 (Cassandra, Dynamo)
- Random Leader Selection: 중복 가능성 있으나 간단

---

## 6. Accuracy (정확성)

- HyperLogLog, Count-min sketch 등으로 근사 처리
- 캐시 정책 관리: TTL, write-through 등
- Eventually Consistent 시스템의 부정확성 고려
- Presto의 query문에는 cardinality 추정 함수가 있음 - 성능은 오르지만 오차 범위내에서만 정확도를 가짐

---

## 7. Complexity / Maintainability (복잡성, 유지보수)

### 최소화 전략
- 공통 컴포넌트 활용: 인증, 캐시, 로깅, TLS 등
- 요구사항 명확화로 과설계 방지
- 구성요소의 분리 및 재사용성 강조
- RPC (protobuf, Thrift)로 메시지 크기 감소

### 운영 유지 전략
- CI/CD: Blue/Green, Rollback 지원
- SonarQube 등 정적 분석 도구 활용
- 장애 분석, Runbook 작성

---

## 8. Cost (비용)

### 트레이드오프 예시
- 수직 확장으로 복잡성 감소 ↔ 비용 증가
- 고가용성 ↔ 더 많은 리소스 소비
- 원거리 서버 사용 ↔ 낮은 인프라 비용

### 고려 항목
- 구현 및 운영 비용
- 기술 교체 및 종료 비용
- 모니터링 및 알림 시스템 유지 비용

---

## 9. Security (보안)

- TLS 종료 vs 데이터센터 내 암호화 유지
- 저장/전송 시 암호화 (At-rest vs In-transit)
- OAuth2, OpenID Connect 이해
- 내부 서비스도 보안 기본 내장 (Zero trust)
- Rate limiting으로 DDoS 방어

---

## 10. Privacy (개인정보 보호)

- PII: 이름, 주소, 계좌번호 등
- 암호화 키 분리 저장 → 키 삭제로 유저 데이터 삭제 효과
- 접근 제어: LDAP, Role-based Access Control
- 데이터 마스킹: 해싱 (SHA-2, SHA-3)
- Audit 로그 필수, 사용자 ID 정기적 변경 고려

---

## 12. Cloud-native

### 핵심 기술
- 컨테이너, 마이크로서비스, Service Mesh, Immutable Infra, Declarative API

### 목표
- 유연하고 관찰 가능하며 자동화된 시스템 구축
- 빠른 변경 대응과 배포 주기 단축

---
## Summary

| 요구사항 | 핵심 내용 요약 |
|----------|----------------|
| Scalability | 수평 확장, 무상태 서비스, Load Balancer |
| Availability | 요청 수락/응답 가능 시간 비율 |
| Fault-tolerance | 일부 컴포넌트 실패에도 운영 지속 |
| Performance | 빠른 응답, 경량화, 위치 최적화 |
| Consistency | Linearizability vs Eventual consistency |
| Accuracy | 근사화 알고리즘, 캐시 정확도 |
| Complexity | 재사용 가능한 공통 컴포넌트 |
| Cost | 비기능 요구사항 간 트레이드오프 |
| Security | TLS, 접근제어, OAuth 등 |
| Privacy | PII 보호, 키 기반 암호화, 접근 로그 |
| Cloud-native | 자동화, 마이크로서비스, 탄력성 |

