# Craigslist 시스템 설계
## 1. 사용자 요구사항
   사용자 유형
   게시자 (Poster): 글 작성, 삭제, 갱신 (7일마다), 자신의 게시물 검색

열람자 (Viewer): 도시별 게시물 검색, 필터, 상세 조회, 이메일로 연락, 신고 가능

게시물 속성
제목, 설명, 가격(단일 통화), 위치 정보

사진 최대 10장 (각 1MB)

향후 비디오 기능 추가 가능

비기능 요구사항
확장성: 도시당 1,000만 사용자 지원

가용성: 99.9% uptime

성능: 게시 직후 몇 초 내 조회 가능, P99 ≤ 1초

보안: 로그인 후 작성 가능 (OpenID Connect 등 사용)

## 2. API 및 데이터 모델
   API 예시
   ```
   POST /post         // 게시물 생성
   PUT /post          // 게시물 수정
   GET /post?id=123   // 상세 조회
   GET /post?search=... // 검색
   DELETE /post/{id}  // 삭제
   POST /contact      // 게시자에게 연락
   POST /report       // 신고
   SQL 스키마 요약
   User: id, 이름, 가입 시각
   ```

Post: id, 게시자 ID, 제목, 설명, 가격, 위치 정보 등

Images: post_id, image_address (object store URL)

Report: post_id, user_id, 신고 유형, 메시지

## 3. 아키텍처 설계
###  3.1 단일 모놀리식 구조
![MonolithDesign.png](MonolithDesign.png)
* 장점
  * 구현 간단
  * 유지보수 비용 낮음

* 단점
  * HTML 중복 저장
  * 모바일 앱 백엔드 재사용 어려움
  * HTML 파싱 기반 분석 필요

### 3.2 전통적 모듈화 구조
![ModularDesign.png](ModularDesign.png)

* 장점
  * 유연한 확장성
  * 이미지 CDN 활용 가능
  * API 백엔드 재사용 가능

## 4. 확장 전략 및 데이터 흐름
### 4.1 지리 기반 파티셔닝 (GeoDNS)
![GeoRouting.png](GeoRouting.png)
* 사용자의 IP 또는 선택한 도시로 요청 라우팅
* 도시별 DB 파티션 가능

### 4.2 게시물 작성 시퀀스
* 클라이언트가 이미지 직접 업로드

![PostUploadClient.png](PostUploadClient.png)

* 백엔드가 이미지 업로드

![PostUploadBackend.png](PostUploadBackend.png)

## 5. 유지보수 및 고급 기능
### 5.1 게시물 삭제
   cron으로 주기적 삭제

DELETE /old_posts 사용

### 5.2 Kafka + Consumer로 쓰기 트래픽 버퍼링
![KafkaWriteScale.png](KafkaWriteScale.png)
* 고성능 쓰기 지원
* Eventually Consistent

### 5.3 검색 및 추천
* Elasticsearch 인덱싱
* 태그 기반 분석 및 추천 가능
* 저장된 검색어 기반 알림

## 6. 기타 고려사항
* 캐시 & CDN
* Redis로 인기 게시물 캐싱
* 이미지 CDN 활용 (CloudFront 등)
* 알림 서비스
* 게시물 갱신 알림 이메일 전송
* 배치 ETL로 매일 갱신 대상 조회

* 태그 및 카테고리

![TagSchema.png](TagSchema.png)

* 저장된 검색어 알림
* 매일 일괄 처리 (ETL)

* 중복 검색어 제거 후 Elastic 호출 → 사용자 알림

### 요약
| 항목	| 설명                            |
|-------------------------------------|-----------------------------|
| 시스템 유형 | 	읽기 중심 (Read-heavy)         |
| 주요 스토리지 | 	SQL + Object Store         |
| 확장 전략 | 	GeoDNS, Cache, Kafka, 파티셔닝 |
| 핵심 속성 | 	단순함, 유지보수 용이성, 저비용         |
| 추가 고려사항 | 	법규 준수, 구독 알림, 정기 삭제, 보고 기능 |