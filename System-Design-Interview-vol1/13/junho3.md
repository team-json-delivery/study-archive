# 13장 검색어 자동완성 시스템

## 1단계 문제 이해 및 설계 범위 확정

- 사용자가 입력하는 단어의 자동완성될 검색어의 위치. 첫 부분인가? 중간 부부인가?
- 몇 개의 자동완성 검색어가 표시되야 하는가?
- 자동완성 검색어 5개를 고르는 기준은 무엇인가?
- 맞춤법 검사 기능을 제공해야 하는가?
- 질의는 영어인가?
- 대문자나 특수 문자 처리도 해야 하는가?
- 얼마나 많은 사용자를 지원해야 하는가?

### 요구사항

- 빠른 응답 속도: 100밀리초 이내여야 한다.
- 연관성
- 정렬
- 규모 확장성
- 고가용성

### 개략적 규모 추정

- 검색창에 글자를 입력할 때마다 클라이언트는 백엔드에 요청을 보낸다.
- 최대 48,000 QPS

## 2단계 개략적 설계안 제시 및 동의 구하기

- 데이터 수집 서비스: 사용자가 입력한 질의를 수집하는 서비스
- 질의 서비스: 주어진 질의에 5개 인기 검색어를 제공하는 서비스

### 데이터 수집 서비스

검색어와 횟수를 저장하는 테이블이 존재하고, 검색어를 입력할 때마다 테이블에 기록

### 질의 서비스

검색어와 횟수를 저장하는 테이블이고, 횟수로 정렬  
LIKE 검색어로 데이터를 조회하지만, 데이터가 많을 경우 성능 저하  

## 3단계 상세 설계

- 트라이 자료구조
- 데이터 수집 서비스
- 질의 서비스
- 규모 확장이 가능한 저장소
- 트라이 연산

### 트라이 자료구조

검색어의 글자 하나하나를 트리구조로 대응  
시간 복잡도 = O(p) + O(c) + O(clogc)  
p: 접두어의 길이  
c: 주어진 노드의 자식 노드 개수

최악의 경우 전체 트라이를 검색해야 하는 일이 생길 수 있음  

> 광고 영역의 키(uuid) 값으로 디렉토리를 만들고, 디렉토리 내부에 광고 캐시파일을 위치 시켰음  
> 맵 구조로 특정 영역에서 요청이 들어오면, 영역의 키값에 해당하는 디렉토리를 바로 접근하기 위함  
> 영역이 늘어날수록 리눅스 디렉토리 개수 제한으로 성능 저하 발생  
> 디렉토리를 트라이 구조 형식으로 변경하자는 논의가 있었음  
> 공수와 영향도가 있을 것으로 판단하여, 사용하지 않는 영역(디렉토리)를 제거하기로 결정  

#### 접두어 최대 길이 제한

#### 노드에 인기 검색어 캐시

### 데이터 수집 서비스

- 매일 수천만 건의 질의가 입력될 때마다 트라이를 갱신하면 성능 이슈 발생
- 트라이가 만들어지고 나면 인기 검색어는 자주 변경되지 않을 것이고, 트라이를 자주 갱신할 필요가 없음  

서비스 성격에 따라 트라이 갱신 빈도가 결정 됨  

#### 데이터 분석 서비스 로그

#### 로그 취합 서버

> Nginx 로그를 수집하여 영역별 광고 노출수를 집계 했음  
> /imp?areaIdx=A10000&adIdx=B10000 형식의 Nginx access.log가 쌓이고, 10분마다 크론탭으로 access_202308230910.log 형식으로 로그 취합서버로 전송  
> 로그 취합 서버에서는 로그 파일을 읽어서 영역별, 광고별로 집계하여 DB와 AWS Athena에 적재  

#### 취합된 데이터

#### 작업 서버

#### 트라이 캐시

#### 트라이 데이터베이스

트라이 데이터베이스는 지속성 저장소다.  
- 문서 저장소: 몽고DB
- 키-값 저장소

#### 질의 서비스

최적화 방안
- Ajax 요청
- 브라우저 캐싱
- 데이터 샘플링

### 트라이 연산

#### 트라이 생성

#### 트라이 갱신

- 매주 한 번 갱신
- 트라이의 각 노드를 개별적으로 갱신: 성능이 좋지 않음

#### 검색어 삭제

API 서버와 트라이 캐시 사이에 필터를 만들어서 부적합한 단어 제외

### 저장소 규모 확장

샤딩  
글자별로 단어의 개수가 차이가 있어서 데이터를 각 서버에 균등하게 배분하는 것은 불가능  
샤드 관리자 서비스를 두고, 관리자 서비스에서 어느 샤드에 있는지 정보를 내려주도록 설계 해야 함  

## 4단계 마무리

- 다국어는 유니코드로 지원
- 국가별 인기 검색어 차이는 CDN으로 대응
- 실시간 대응은 현재 설계안으로는 불가능