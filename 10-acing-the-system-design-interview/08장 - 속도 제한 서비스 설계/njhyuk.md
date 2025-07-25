# 8장 속도 제한 서비스 설계
## 8.1 속도 제한 서비스의 대안과 그것이 실현 불가능한 이유
* 오토 스케일링
  * 트래픽 급증이 감지될 때 새 호스트 추가 과정이 너무 느릴 수 있음
  * 처리하지 말아야할 악의적 요청 처리
* 악의적인 요청을 처리해서는 안됨
  * 속도 제한은 이러한 요청의 IP 주소를 감지하고 버림으로 방어
* 속도 제한이 별도 서비스여야 하는 이유
  * 특정 요청이 그 외 요청보다 비용이 더 많이 들기 때문
* 7계층 로드 밸런서 사용
  * 비용이 많이 드는 요청을 거부할 수 있지만, 비용과 복잡성이 높음
## 8.2 속도 제한을 하지 말아야 할 때
* 사용자가 너무 많은 구독 요청, Too Many Request 를 반환하면 사용자 경험이 좋지 않음
* 요청 속도에 다라 구독 요금을 부과하는 서비스
  * 속도 제한 서비스는 각 클라이언트에 다른 속도 제한을 주는것과 같은 복잡한 사용 사례가 아닌 단순한 사용 사례에 국한되어야 한다
## 8.3 기능적 요구사항
* 회사 외부이용자가 이용함
* 최대 요청 속도를 설정할 수 있어야 하며, 이를 초과하는 동안 429 응답과 거부
## 8.4 비기능적 요구사항
* 확장성
  * 일일 수십억건의 요청으로 확장할 수 있어야함
  * 서비스에 10억명 사용자, 사용자당 100개의 요청 저장, 각각 64비트, 보존기간 10초
    * 100 * 64 * 101 * 10M = 808GB
    * 레디스에 사용자당 키를 할당하면 64 * 100 = 800바이트
* 성능
  * 속도 제한기 요청의 응답시간이 사용자 요청의 응답시간에 추가됨
  * P99 100ms
* 복잡성
  * 속도 제한기로써 단일 기능 집중, 비용 최소화, 단순해야함
* 보안과 프라이버시
  * 다른 사용자 서비스의 요청을 위조해 속도를 제한하려는 방식으로 공격
  * 다른 사용자 서비스의 속도 제한기 요청자 데이터를 요청
* 가용성과 내결함성
  * 고가용성이나 내결함성이 필요하지 않을 수 있다, 그동안 속도제한을 하지 않을 수 있음
* 정확성
  * 클라이언트를 잘못 식별해 속도를 제한해서는 안됨
* 일관성
  * 몇초간의 불일치는 허용될 수 있음, 최종 일관성 허용
## 8.5 사용자 스토리와 필요한 서비스 구성 요소
* 사용자 ID와 사용자 서비스 ID가 필수로 포함
* 속도 제한기는 위 데이터를 60초 동안 저장해야함, 인메모리 저장소에 저장이나 캐싱
* 사용자 서비스는 자신의 엔드포인트 속도 제한을 생성하고 업데이트 하기 위해 속도 제한 서비스에 요청 할 수 있음
* 요구사항 종합
  * 카운트에 빠른 읽기 쓰기가 가능한 데이터베이스, 레디스와 같은 인메모리 데이터베이스
  * 규칙을 정의하고 검색할 수 있는 서비스
  * 규칙 서비스와 레디스 데이터베이스에 요청을 보내는 서비스
## 8.6 고수준 아키텍처
* 백엔드의 단계
  1. 규칙서비스에서 서비스의 속도 제한을 가져온다.
  2. 이 요청을 포함한 서비스의 현재 요청 속도를 결정한다.
  3. 요청 속도 제한 여부를 나타내는 응답을 반환한다.
* 규칙 서비스 사용자는 읽기와 쓰기 모두 포함하는 SQL 쿼리를 리더 노드에 수행, 백엔드는 읽기쿼리만 팔로워 노드에 수행해야 한다. (높은 일관성, 높은 성능 경험)
* 규칙이 자주 변경되지 않을것으로 예상되므로 규칙 서비스에 레디스 캐시를 추가해 읽기 성능 향상할 수 있음
## 8.7 상태 저장 접근 방식/샤딩
* 상태 저장 접근 방식
  * 요청이 도착하면 로드 밸런서가 해당 호스트로 라우팅 한다.
  * 각 호스트는 자신의 클라이언트 수를 메모리에 저장한다.
  * 호스트는 사용자가 속도 제한을 초과했는지 판단한다.
* 상태 저장 접근 방식은 7계층 로드 밸런서가 필요하다.
* 호스트가 다운되면 새 호스트로 라우팅 해야 한다.
* 핫 샤드를 모니터링 하고 주기적으로 호스트간 트래픽을 재조정 해야한다.
  * ETL 작업으로 주기적으로 재조정을 할 수 있다.
* 상태 저장 접근 방식은 더 복잡하고 일관성과 정확성이 높지만 다음 항목은 더 낮다.
  * 비용
  * 가용성
  * 내결함성
## 8.8 모든 호스트에 모든 카운트 저장
### 8.8.1 고수준 아키텍처
* 접근방식
  * 속도 제한 결정을 내리고 이를 반환한다.
  * 비동기적으로 다른 호스트와 타임스탬프를 동기화 한다.
* 4계층 로드밸런서가 요청을 무작위로 분배한다.
  * 호스트간 속도 제한을 동기화 해야 한다.
  * 이 방식에선 배치 대신 스트리밍 방식을 사용한다.
* 트레이드 오프
  * 일관성과 정확성 대신 낮은 지연시간 및 높은 성능을 얻는다.
* 호스트가 다운되고 데이터가 손실되면
  * 더 많은 요청이 가능하지만 허용 가능하다.
### 8.8.2 카운트 동기화
* 동기화 매커니즘 - 풀 vs 푸시
  * 일관성과 정확성 대신 더 높은 성능 낮은 리소스 낮은 복잡돌르 위해 푸시 방식 선택
* 호스트가 다운되면 해당 카운트를 무시하고 더 많은 요청할 수 있음
* 이러한 고려사항을 바탕으로 TCP 대신 UDP를 사용해 비동기적으로 타임스탬프 공유하게 결정할 수 있음
#### 전체 대 전체
* 그롭 내 모든 노드가 다른 모든 노드에 메시지를 전송하는 방식
* 네트워크의 모든 모드가 다른 노드와 연결된 완전 메시를 필요로함
* 확장 가능하지 않음. 노드 수에 따라 제곱으로 호가장됨.
#### 가십 프로토콜
* 노드가 주기적으로 무작위로 다른 노드를 선택해 메시지를 보냄
* 일관성과 정확성을 낮춰 더 높은 성능과 낮은 리소스 소비, 단, 더 복잡함
#### 외부 저장소와 조정 서비스
* 호스트는 리더 호스트를 통해 서로 통신
* 주키퍼와 같은 클러스터의 구성 서비스에 의해 선택
* 리더 호스트의 IP 주소만 알면 되고, 리더 호스트는 주기적으로 호스트 목록 업데이트
#### 무작위 리더 선출
* 리더 선출 알고리즘을 사용해 복잡성을 낮추는 대신 더 높은 리소스 소비 선택
* 여러 리더가 선출되어 메시징 오버헤드가 있을수 있지만, 감안할 수 있음
## 8.9 속도 제한 알고리즘
### 8.9.1 토큰 버킷
버킷은 세가지 특성을 가짐
* 최대 토큰 수
* 현재 사용 가능한 토큰 수
* 버킷에 토큰이 추가되는 보충 속도
매커니즘
* 요청이 도착할때 마다 토큰을 하나 제거한다
* 매초 토큰이 10개 미만이면 1개 추가한다
구현
* 해시맵 키-값쌍
* 호스트에 사용자 ID 키가 없으면 토큰 카운트 9로 초기화
* 있으면 토큰 감소
* 카운트가 0이면 속도 제한
이해하고 구현하기 쉽고 메모리 효율적
### 8.9.2 누수 버킷
* 최대 토큰 수가 있고, 고정 속도로 누수되며, 비어있을때 누수가 멈춘다. 요청이 도착할때마다 버킷에 토큰을 추가한다. 버킷이 가득차면 요청이 거부되거나 속도가 제한된다.
* 구현
  * 고정 크기의 FIFO 큐를 사용한다
  * 큐는 주기적으로 디큐된다
  * 요청이 도착하면 인큐된다
  * 고정된 큐 크기이므로 메모리 효율성이 낮다
* 문제점
  * 매초 호스트는 모든 키의 모든 큐를 디큐해야 한다.
  * 오래된 키를 삭제하는 별도의 매커니즘이 필요하다.
  * 큐는 용량을 초과할 수 없으므로 분산 구현에서는 여러 호스트가 동기화 하기 전에 동시에 그 큐를 완전히 채울 수 있다.
### 8.9.3 고정 윈도우 카운터
* 매커니즘
  * 키는 클라이언트 ID + 타임스탬프
  * 값은 요청 횟수
  * 클라이언트가 요청하면 키가 존재하면 증가하고 존재하지 않으면 생성
  * 윈도우 간격은 고정, ex) 각 분의 0~60초 사이
  * 윈도우가 지나면 모든 키가 만료된다.
* 단점
  * 속도 제한 경계를 이용해 속도 제한의 최대 2배까지 요청 속도를 허용할 수 있다.
### 8.9.4 슬라이딩 윈도우 로그
* 각 클라이언트 키-값 쌍으로 구현된다.
* 키는 클라이언트 ID, 값은 정렬된 타임스탬프 목록
* 매커니즘
  * 새로운 요청이 들어오면 해당 타임스탬프를 추가하고 첫번째 타임스탬프가 만료됐는지 확인한다.
  * 만료됐다면 이진검색을 수행해 마지막으로 만료된 타임스탬프를 찾은 다음, 그 이전의 모든 타임스탬프를 제거한다.
  * 큐는 이진검색을 지원하지 않으므로 큐 대신 리스트를 사용한다.
  * 리스트에 10개 이상의 타임스탬프가 있으면 참을 반환하고, 그렇지 않으면 거짓을 반 환한다.
* 슬라이딩 윈도우 로그는 정확하다. 그러나 모든 요청에 타임스탬프를 저장하는 것은 많은 메모리를 소비한다.
### 8.9.5 슬라이딩 윈도우 카운터
* 여러개의 고정 윈도우 간격을 사용하며, 각 간격은 속도 제한 시간 윈도우의 1/60이다.
* 예를들어 속도 제하 간격이 1시간이면 1시간짜리 윈도우 하나 대신 1분짜리 윈도 60개를 사용한다.
* 현재 속도는 마지막 60개 윈도우를 합산해 결정한다.
## 8.10 사이드카 패턴 적용
* 관리자는 제어 평면에서 사용자 서비스의 속도 제한 정책을 구성할 수 있으며 이는 사이드카 호스트로 배포된다.
* 사용자 서비스가 사이드카 호스트에 속도 제한 정책을 포함하는 이 설계를 통해 사용자 서비스 호스튼 속도 제한 정책을 조회하기 위해 속도 제한 서비스에 요청을 보낼 필요가 없어 네트워크 오버헤드를 줄일 수 있다.
## 8.11 로깅, 모니터링, 경보
* 밴을 당했음에도 불구하고 계속해서 높은 비율로 요청하는 악의적 활동 징후
* 짧은 시간 내에 비정상적으로 많은 수의 사용자가 속도 제한을 받는 것과 같은 잠재적 디도스 징후
## 8.12 클라이언트 라이브러리로 기능 제공
* 서버 어플리케이션은 오랫동안 구 버전 클라이언트를 계속 지원해야 한다.
* 클라이언트에서 데이터 처리를 하고자 한다면 다음 조건중 최소한 하나는 충족 해야 한다.
  * 처리가 단순해 서버 어플리케이션의 향후 버전에서도 이 클라이언트 라이브러리를 쉽게 계속 지원할수 있어야 한다.
  * 처리가 리소스 집약적이어서 클라이언트에서 이러한 처리를 수행하는 유지보수 부담이 서비스 운영 비용의 상당한 절감과 맞바꿀만한 가치가 있어야 한다.
  * 클라이언트가 더이상 지원되지 않을 시기를 사용자에게 명확히 알리는 수명 주기가 있어야 한다.
* 클라이언트가 직접 속도 제한 서비스를 구현
  * 클라이언트끼리 통싱하지 않으므로, 특정 호스트의 요청 속도만 측정할 수 잇는 문제가 있다.
