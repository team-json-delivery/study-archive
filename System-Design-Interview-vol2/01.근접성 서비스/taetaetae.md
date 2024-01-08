# 1장 근접성 서비스

## 1단계 : 문제 이해 및 설계 범위 확정
- 기능 요구사항
  - 사용자의 위치(경도/위도)와 검색 반경 정보에 매치되는 사업장 목록 반환
  - 사업장 소유주가 사업장 정보 관리, 실시간 반영 x
  - 사업장의 상세 정보 조회
- 비기능 (non-functional requirements) 요구사항
  - 낮은 응답지연 (latency)
  - 데이터 보호 (data privacy)
  - 고가용성 (high availability) 및 규모 확장성(scalability)
- 개략적인 추정 (back of the envelope calculation)
  - 일간 능동 사용자(Daily Active User, DAU) : 1억명
  - 등록된 사업장 수 : 2억
  - 1일 = 24시간 x 60분 x 60초 = 86,400초 (대략 100,000)
  - 한 사용자당 하루에 5회 검색
  - QPS = (1억명 x 5회) / 100,000 = 5,000