# Chapter 6: Common Services for Functional Partitioning

## 1. 개요

- 기능적 분할(Function Partitioning)은 백엔드 서비스 중 공통 기능을 분리하여 별도의 공통 서비스로 구성하는 확장성 기법.
- API Gateway, Service Mesh/Sidecar, Metadata Service 등의 도입을 통해 중복 구현 제거, 일관된 처리, 독립적 확장이 가능.
- 또한, REST/RPC/GraphQL 등의 API 패러다임 선택, 라이브러리 vs. 서비스 구현 고려, 다양한 프레임워크 사용에 대한 논의도 포함됨.

---

## 2. API Gateway

### 2.1 공통 기능의 중앙 집중화
- 다양한 서비스에서 중복되는 보안, 에러 처리, 성능 최적화 기능을 API Gateway에 모음.
- 주요 예: Amazon API Gateway, Kong 등.

### 2.2 기능별 역할
- **보안**: 인증(Authentication), 권한 부여(Authorization), TLS 종료, 서버 측 암호화.
- **에러 처리**: 요청 유효성 검사, 중복 요청 제거.
- **성능 및 가용성**: 캐싱, 레이트 리미팅, 요청 디스패칭, Circuit Breaker.
- **로깅 및 분석**: 실시간 요청 로깅, 분석, 디버깅을 위한 데이터 수집.

---

## 3. Service Mesh / Sidecar

- API Gateway의 단점(지연, 중앙 집중 자원 부담)을 보완하기 위한 대안.
- 각 서비스에 프록시(Envoy 등)를 사이드카로 붙여 요청을 중계하고 보안/모니터링 등을 수행.

### 3.1 구성 요소
- **Control Plane**: 인증서, IAM, 설정 전파.
- **Data Plane**: Envoy 프록시 간 통신.
- **Observability Plane**: 로깅, 모니터링, 트레이싱 (Jaeger, Prometheus 등).

---

## 4. Metadata Service

- 시스템 내 여러 컴포넌트에서 참조되는 데이터를 메타데이터 서비스로 분리 저장.
- 큐나 메시지에 전체 객체 대신 ID만 포함 → 중복 전송/저장 방지.
- 단점: 복잡성 증가, 레이턴시 증가, 서비스 부하 고려 필요.

---

## 5. Service Discovery

- 서비스 간 통신을 위한 주소/포트 관리 시스템.
- 클라이언트 또는 서버 측 탐색 (client-side/server-side discovery).
- 보통 인프라 팀에서 관리하며 일반 엔지니어는 상세 내용 몰라도 무방.

---

## 6. 프레임워크 및 플랫폼

### 6.1 시스템 구성 요소별 프레임워크 예시

#### Web (Browser)
- React, Vue.js, Angular
- Transpile 기반 언어: TypeScript, Elm, ClojureScript 등

#### Mobile
- **Native**: Kotlin (Android), Swift (iOS)
- **Cross-platform**: React Native, Flutter, Ionic, Xamarin

#### Backend
- **REST**: Spring Boot, Flask, Django, Rails
- **RPC**: gRPC, Thrift, Avro
- **GraphQL**: Apollo Server 등

#### Server-side Web
- Express (Node.js), Deno, Rocket (Rust), Vapor (Swift)

### 6.2 Web Server App의 역할
- 웹 요청 라우팅 및 백엔드 집계 역할 수행.
- 브라우저가 직접 백엔드와 통신 시 불필요한 데이터 전송이 많아 Node.js 등 서버앱을 중간에 둠.
- REST가 불편한 복잡 쿼리 상황에서는 GraphQL이 대안이 될 수 있음.

---

## 7. 라이브러리 vs. 서비스

| 항목 | 라이브러리 | 서비스 |
|------|-------------|----------|
| 배포/버전 관리 | 사용자 관리 | 중앙 관리 |
| 확장성 | 사용자 측에서 확장 | 독립 확장 가능 |
| 언어 의존성 | 있음 | 없음 |
| 레이턴시 | 낮고 예측 가능 | 네트워크에 따라 가변 |
| 보안 | 코드 노출 가능 | 코드 노출 없음 |
| 디버깅 | 어렵고 환경 다양 | 로그/환경 제어 가능 |

---

## 8. API Paradigms

### 8.1 REST
- HTTP 기반, stateless, 쉬운 학습과 디버깅.
- **장점**: 캐싱, 하이퍼미디어(HATEOAS), OpenAPI 문서화 지원.
- **단점**: 표준 부족, 문서화 필요, 데이터 과다 또는 부족 전송 문제.

### 8.2 RPC
- 원격 프로시저 호출 방식. (gRPC, Thrift 등)
- **장점**: 바이너리 프로토콜, 효율적 네트워크 사용, 스키마 기반 문서화.
- **단점**: 디버깅 어려움, 외부와의 호환성 문제.

### 8.3 GraphQL
- 클라이언트가 필요한 데이터만 선택적으로 요청.
- **장점**: 정확한 데이터 요청, API 문서 자동화.
- **단점**: 보안 이슈, 분석 어려움, 러닝 커브.

### 8.4 WebSocket
- TCP 기반의 상시 연결 (stateful).
- **장점**: 양방향 통신, 낮은 지연.
- **단점**: 확장성 낮음, 상태 유지 비용 발생.

---

## 9. 요약

- API Gateway는 보안, 성능, 로깅 등 공통 기능을 집중화한 경량 서비스.
- Service Mesh는 API Gateway의 대체 수단으로 서비스 간 통신 제어를 프록시 사이드카에 위임.
- Metadata Service는 중복 제거 및 정규화를 위한 ID 기반 조회 구조.
- 프레임워크는 용도에 따라 다양하게 선택 가능하며, Node.js와 같은 웹 서버 앱이 중간 역할을 수행.
- 라이브러리는 낮은 레이턴시와 언어 의존성이 있지만 버전 관리와 보안에 단점.
- API는 REST, RPC, GraphQL, WebSocket 중 목적에 따라 선택하며, 각각의 장단점을 고려해야 함.

