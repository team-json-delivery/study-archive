# DynamoDB로 Blog + Comment 시스템 만들기: 단일 테이블 설계와 전통적인 DB 설계 비교

> 작성일: 2025-05-25

---

## 📌 문제 정의

우리는 다음과 같은 요구사항을 가진 블로그 시스템을 DynamoDB로 구축하고자 했다.

- 특정 **카테고리(category)**의 블로그 글을 **최신순**으로 조회
- 각 블로그 글에 달린 **댓글(Comment)** 들을 **함께 가져오기**
- **한 번의 Query API 호출**로 블로그 글과 댓글을 모두 가져오기
- **페이징 지원**
- **30일 TTL(Time-to-Live)** 적용으로 데이터 자동 만료

---

## 🧱 전통적인 MySQL 설계 방식

MySQL에서라면 다음과 같은 방식으로 정규화된 테이블을 구성했을 것이다:

```sql
-- blog 테이블
CREATE TABLE blog (
  id BIGINT PRIMARY KEY,
  category VARCHAR(50),
  title TEXT,
  author VARCHAR(100),
  created_at DATETIME
);

-- comment 테이블
CREATE TABLE comment (
  id BIGINT PRIMARY KEY,
  blog_id BIGINT,
  content TEXT,
  author VARCHAR(100),
  created_at DATETIME,
  FOREIGN KEY (blog_id) REFERENCES blog(id)
);
```

**문제점:**

- 카테고리별 블로그를 최신순으로 가져온 후
- 각 블로그 ID로 댓글을 가져와야 하는 **N+1 쿼리 구조**
- **데이터가 수천만 건**으로 증가하면 성능, 운영, 확장 모두 문제 발생

---

## ⚠️ MySQL의 한계 (수천만 건 이상)

| 문제 | 설명 |
|------|------|
| 인덱스 크기 증가 | 성능 저하 유발 |
| 수동 파티셔닝 | 샤딩 및 분산 처리 직접 구현 필요 |
| 수평 확장 한계 | MySQL은 기본적으로 수직 확장을 전제로 함 |
| 실시간 응답성 한계 | 쿼리 복잡도 증가 시 실시간 조회 어려움 |

---

## ⚡ DynamoDB는 무엇이 다른가?

DynamoDB는 전통적인 관계형 모델과 달리 **정규화가 아닌 쿼리 패턴 중심**으로 설계를 해야 한다.  
또한 **JOIN이 없기 때문에** 필요한 데이터를 **한 파티션 안에 함께 저장**하는 방식으로 접근해야 한다.

---

## ✅ 우리가 선택한 단일 테이블 설계

### 📐 테이블 구조

- **PK**: `CATEGORY#<category>`
- **SK**: `<ISO8601 timestamp>#BLOG#<postId>#META`  
           `<ISO8601 timestamp>#BLOG#<postId>#COMMENT#0001` 등

### 예시:

| PK               | SK                                              | type    |
|------------------|--------------------------------------------------|---------|
| CATEGORY#tech     | 2025-05-22T10:00:00Z#BLOG#123#META               | META    |
| CATEGORY#tech     | 2025-05-22T10:00:00Z#BLOG#123#COMMENT#0001       | COMMENT |
| CATEGORY#tech     | 2025-05-22T10:00:00Z#BLOG#123#COMMENT#0002       | COMMENT |

이 구조 덕분에,

* 한 Query로 특정 카테고리의 글과 그 댓글을 함께 가져올 수 있고,
* ISO 8601 포맷 덕분에 **시간 정렬이 자연스럽게 보장**되며,
* 댓글도 연달아 붙어 있기 때문에 **한 번의 조회로 논리적인 하나의 블로그 글 객체를 조립**할 수 있습니다.
---

## 📊 DynamoDB vs MySQL 비교

| 항목                      | MySQL                             | DynamoDB                          |
|---------------------------|-----------------------------------|-----------------------------------|
| 데이터 모델               | 정규화된 관계형 모델              | 비정형 NoSQL 모델                 |
| 확장성                    | 수직 확장 또는 수동 샤딩 필요     | 자동 수평 확장                    |
| 쿼리 성능                 | 복잡한 쿼리에 강점                | 단순 쿼리에 최적화                |
| 스키마 유연성             | 고정된 스키마                     | 유연한 스키마                     |
| 데이터 일관성             | 강한 일관성                       | 기본적으로 최종 일관성            |
| 운영 복잡성               | 인프라 및 스케일링 관리 필요      | 완전 관리형 서비스                |


| 항목           | RDB(MySQL) 방식     | DynamoDB 방식 (Single Table) |
| ------------ | ----------------- | -------------------------- |
| 데이터 모델링 기준   | 데이터의 구조와 관계       | **쿼리 패턴(Query Pattern)**   |
| 정규화 여부       | 최대한 정규화           | **의도적으로 중복 허용**            |
| 관계 표현 방법     | Foreign Key, JOIN | PK + SK 구조로 병렬 저장          |
| 쿼리 성능 최적화 방식 | 인덱스 + 조인 최적화      | PK 단위 Query + SK 설계        |
| 비용 구조        | CPU/메모리 기반 쿼리 비용  | **요청 수 및 용량 기준 과금**        |
| 확장성          | 수직 확장             | **수평 확장 자동 내장**            |
---

## 📦 페이징 구현 방식

```java
QueryRequest request = QueryRequest.builder()
    .tableName("BlogCategoryTable")
    .keyConditionExpression("PK = :category")
    .expressionAttributeValues(Map.of(
        ":category", AttributeValue.fromS("CATEGORY#tech")
    ))
    .limit(1000)
    .scanIndexForward(false) // 최신순 정렬
    .build();
```

- `Limit`은 row(item) 기준
- `LastEvaluatedKey`를 활용한 Cursor 기반 페이징
- 같은 BlogId의 마지막 항목이 끊기면 다음 페이지에서 이어붙이기

---

## 🧠 고려했던 대안들

### ❌ GSI(category + createdAt) 사용

* 글 목록은 정렬해서 조회 가능하지만, 댓글 포함 불가
* 댓글까지 포함하려면 GSI에 모든 댓글까지 인덱싱 → **쓰기 비용 증가, 유지 어려움**

### ❌ Blog와 Comment를 각각 저장 후 N+1 쿼리

* 효율성 문제, 비용 문제

### ❌ Store large blogs in S3 and reference from DynamoDB

* ❌ **Fails to meet latency SLA for real-time access**
* 복잡도 증가 (multi-source hydration 필요)

---

## ✅ 최종 결정의 장점

* 하나의 테이블에 카테고리별 블로그 글과 댓글을 함께 저장
* 시간 순으로 정렬된 SK 덕분에 최신순 정렬도 가능
* 한 Query로 블로그 + 댓글 모두 조회 가능
* TTL도 적용 가능
* 페이징도 `LastEvaluatedKey`로 안정적 처리

---

## 🧠 왜 RDB가 아닌 DynamoDB였을까?

MySQL로도 이 구조를 구현할 수는 있었지만, **수천만 건 이상의 데이터를 다뤄야 하는 경우** 다음과 같은 이유로 DynamoDB가 더 적합하다.

- **자동 확장성** (수평 확장 내장)
- **낮은 지연 시간** (단일 Query로 데이터 조회)
- **페이징과 TTL 지원**
- **운영 복잡도 감소** (인프라 없이 완전 관리형)

---

## ✅ 결론

전통적인 정규화 기반의 MySQL 설계는 관계 표현과 일관성 측면에서 강점을 갖고 있지만,  
데이터 규모가 커지고 읽기 패턴이 단순해질수록 NoSQL, 특히 **DynamoDB의 단일 테이블 설계가 더 효율적인 선택**이 될 수 있다.

이번 설계를 통해 다음을 달성했다:

- **카테고리 기준 최신순 정렬 + 댓글 포함 조회를 단일 쿼리로 수행**
- **애플리케이션 레벨에서 그룹핑하여 블로그 객체 구성**
- **TTL 및 페이징 지원 포함**
- **MySQL 대비 운영, 성능, 확장성에서 실질적 이점 확보**

> 👉 정규화보다 쿼리 패턴 중심의 데이터 모델링이 필요한 시스템이라면, DynamoDB 단일 테이블 설계를 꼭 고려해보자.

