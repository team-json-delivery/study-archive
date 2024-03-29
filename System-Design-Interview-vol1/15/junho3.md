# 15장 구글 드라이브 설계

## 1단계 문제 이해 및 설계 범위 확정

- 파일 업로드 / 다운로드, 파일 동기화, 알림
- 모바일 앱 / 웹 모두 지원
- 파일 암호화 제공
- 파일 크기 최대 10GB 제한
- 일 1,000만명 사용자

비 기능적 요구사항
- 안정성
- 빠른 동기화 속도
- 네트워크 대역폭
- 규모 확장성
- 높은 가용성

### 개략적 추정치

- 필요한 저장공간: 500 페타바이트
- 업로드 API QPS: 240
- 최대 QPS : 480

## 2단계 개략적 설계안 제시 및 동의 구하기

drive/유저아이디/파일 형식으로 디렉토리 설계

### API

#### 1. 파일 업로드 API

- 이어 올리기: 파일 사이즈가 크고 네트워크 문제로 업로드가 중단될 경우를 대비

> TUS https://shanepark.tistory.com/441

#### 2. 파일 다운로드

> path를 그대로 노출하는 것은 위함하다고 생각함  
> 중간 맵핑 테이블을 두고 uuid와 같은 값으로 조회하도록 해야함

#### 3. 파일 갱신 히스토리 API

### 한 대 서버의 제약 극복

N 대의 서버를 두고, % N으로 샤딩하는 방법으로 해결할 수 있지만, 단기적인 방법임  
장기적으로는 S3와 같은 저장소를 활용해야 함

### 동기화 충돌

> Git Merge

### 개략적 설계안

## 3단계 상세 설계

### 블록 저장소 서버

용량이 큰 파일이 업데이트가 발생했을 때 전체 파일을 서버로 보내면 네트워크 대역폭을 많이 잡아먹어 최적화가 필요함  
- 델타 동기화: 수정일 일어난 블록만 동기화
- 압축: 블록 단위로 압축해 두면 데이터 크기를 많이 줄일 수 있다.  

### 높은 일관성 요구사항

이 시스템은 강한 일관성 모델을 기본으로 지원해야 한다.  
같은 파일이 단말이나 사용자에 따라 다르게 보이면 안 됨  
관계형 데이터베이스를 채택하여 ACID를 보장해야 함  

### 메타데이터 데이터베이스

### 업로드 절차

### 다운로드 절차

### 알림 서비스

파일의 일관성을 유지하기 위해, 클라이언트는 로컬에서 파일이 수정되었음을 감지하는 순간 다른 클라이언트에 알려서 충돌을 막아야 함  

- 롱 폴링
- 웹소켓

서비스 특성상 롱 폴링이 적합함  
웹소켓을 사용해서 양방향 통신을 사용할 필요가 없음  

### 저장소 공간 절약

- 중복 제거
- 지능적 백업 전략 도입
  - 보관해야 하는 파일 버전 개수에 제한을 두고, 제일 오래된 버전 삭제
  - 중요한 버전만 보관
- 아카이빙 저장소 활용

### 장애 처리

- 로드밸런서 장애
- 블록 저장소 서버 장애
- 클라우드 저장소 장애
- API 서버 장애
- 메타데이터 캐시 장애
- 메타데이터 데이터베이스 장애
- 알림 서비스 장애
- 오프라인 사용자 백업 큐 장애

## 4단계 마무리


