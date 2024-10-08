# 6장 서버 메트릭

MySQL 메트릭은 성능과 밀접한 관련이 있습니다.  
다만, 이것들이 어떻게 관련되는지는 분명하지 않습니다.  

MySQL 메트릭을 자주 다루기는 하지만 제대로 배운 적은 없으므로 블랙박스로 간주하더라도 그리 틀리지는 않을 겁니다.  
MySQL 메트릭 참고 자료나 문서가 부족한 이유는 메트릭이 나타내는 바가 자명해서 이해하거나 해석할 필요가 없다는 잘못된 가정 때문입니다.  

메트릭은 워크로드가 MySQL이라는 프리즘을 통과하면서 굴절되어 드러나는 스펙트럼입니다.  

- MySQL에서 워크로드를 실행하면 메트릭 결과는 MySQL이 아닌 워크로드의 속성을 나타냅니다.
- 성능은 쿼리, 데이터, 접근 패턴과 같이 워크로드에 직접적인 영향을 받습니다.

MySQL 서버 성능을 이해하고 분석하는 데 필요한 메트릭은 극히 일부에 불과합니다.  
나머지 메트릭의 관련성과 중요성은 다양합니다.  

- 일부는 잡음입니다.
- 일부는 더는 사용하지 않습니다.
- 일부는 기본적으로 비활성화되어 있습니다.
- 일부는 기술적으로 매우 구체적입니다.
- 일부는 특정한 경우에만 유용합니다.
- 일부는 정보 제공용이며 적절한 메트릭이 아닙니다.
- 일부는 사람이 이해하기 어렵습니다.


## 6-1 쿼리 성능 대 서버 성능

MySQL 성능에는 쿼리 성능과 서버 성능이라는 2가지 측면이 있습니다.

- 쿼리 성능: 워크로드 최적화
- 서버 성능: 워크로드를 처리하는 MySQL 성능

입력은 워크로드이고 출력은 서버 성능입니다.  
최적화된 워크로드를 MySQL에 넣으면 성능이 향상됩니다.  

### 동시성과 경합

동시성은 쿼리 성능을 떨어뜨리는 경합으로 이어집니다.  
단독으로 실행되는 쿼리는 다른 쿼리와 함께 실행될 때 다른 성능을 냅니다.  

서버 성능 분석은 모든 쿼리가(동시성) 공유되고 제한된 시스템 리소스를 놓고 경쟁할 때 MySQL이 워크로드를 처리하는 방식을 확인하는 데 가장 유용하고 일반적으로 수행됩니다.  

### 튜닝

서버 성능에는 MySQL, 운영체제, 하드웨어 등 3가지 추가 요소가 있습니다.  

튜닝을 해야 한다면 알려진 안정적인 워크로드로 서버 성능을 분석해야 합니다.  
그렇지 않으면 성능 향상이 튜닝의 결과라고 확신할 수 없습니다.  
튜닝 결과는 제어, 변수, 재현성, 반증 가능성과 같은 기초 지표를 이용하여 과학적으로 측정할 수 있습니다.  

### 성능 회귀

일반적으로 쿼리 성능, MySQL 튜닝, 하드웨어 결함이 문제가 아님을 확인한 전문가는 최후 수단으로 성능 회귀(또는 버그)를 의심합니다.  

튜닝과 성능 회귀는 MySQL DBA와 전문가의 책임입니다.  


## 6-2 정상과 안정

인간은 패턴 인식에 능숙해서 어떤 메트릭을 보여주는 차트가 비정상인지 쉽게 알 수 있습니다.  

### 정상

정상은 성능의 일부 측면이 평소보다 더 높거나 낮은지, 더 빠르거나 느린지, 더 좋거나 나쁜지를 결정하는 기준선입니다.  

### 안정

더 나은 성능을 추구하는 과정에서 안정적인 성능을 놓쳐서는 안 됩니다.  
한계에서 성능이 불안정해지면 성능보다 더 큰 문제가 발생합니다.  


## 6-3 핵심 성능 지표

### 응답 시간

쿼리 응답 시간이 MySQL 성능의 핵심이라 설명했듯이 응답 시간은 모두가 관심을 두는 유일한 지표입니다.  
그러나 응답 시간이 빠르더라도 다른 핵심 성능 지표도 함께 고려해야 합니다.  

### 오류

기본적은 쿼리 오류뿐만 아니라 쿼리, 연결, 클라이언트, 서버와 같은 모든 오류를 포함합니다.  

### QPS

QPS는 성능을 나타내지만 그 자체가 성능은 아닙니다.  

### 실행 중인 스레드

실행 중인 스레드는 MySQL이 QPS를 달성하기 위해 얼마나 노력하는지를 측정합니다.  
목표는 정상이고 안정적인 스레드 실행, 즉 낮을수록 좋습니다.  

응답 시간, 오류율, QPS, 실행 중인 스레드는 항상 모니터링하세요.  


## 6-4 메트릭 필드

메트릭 필드는 메트릭이 어떻게 관련되어 있는지 이해하기 위한 모델입니다.  

### 응답 시간 메트릭

응답 시간 메트릭은 MySQL이 응답하는 데 걸리는 시간을 나타냅니다.  
하위 수준의 세부 사항을 포함하므로 메트릭 필드에서 최상위 수준입니다.  

### 속도 메트릭

속도 메트릭은 MySQL이 개별 작업을 얼마나 빨리 완료하는지를 나타냅니다.  
초당 쿼리 수(QPS)는 어디에나 있으며 보편적으로 알려진 데이터베이스의 속도 메트릭입니다.  

전반적으로 QPS가 증가하면 쿼리가 많을수록 CPU 시간이 더 많이 필요하므로 CPU 사용률이 증가할 수 있습니다.  
CPU 사용률이 증가하는 것을 방지하거나 줄이려면 쿼리를 최적화하여 실행하는 데 필요한 CPU 시간을 줄여야 합니다.  

### 사용률 메트릭

사용률 메트릭은 MySQL이 유한한 리소스를 얼마나 많이 사용하는지를 나타냅니다.  
- CPU 사용량
- 메모리 사용량
- 디스크 사용량

**제한 속도**는 사용률로 표현할 수 있습니다.  
디스크 I/O 사용률은 최대 속도 대비 현재 속도입니다.  
**무한 속도**는 최대 속도가 없기 때문에 사용률로 표현할 수 없습니다.  

사용률이 높아지면 관련 속도가 낮아질 수 있습니다.  

사용률이 100%에 가까워지면 MySQL은 대기합니다.  

부하를 줄이거나(워크로드 최적화) 하드웨어 용량을 늘려 개선할 수 있습니다.  
회전 디스크를 NVMe로 업그레이드하면 스토리지 대기 시간이 크게 줄어듭니다.  

### 대기 메트릭

대기 메트릭은 쿼리 실행 중 유휴 시간을 나타냅니다.  
경합과 일관성으로 인해 쿼리 실행이 지연되면 대기가 발생합니다.  
대기 메트릭은 속도나 응답 시간으로 계산하지만, MySQL이 작동하지 않을 때를 나타내므로 별도로 분류할 가치가 있습니다.  

대기는 피할 수 없습니다.  
목표는 대기를 없애는 것이 아니라 줄이고 안정시키는 것입니다.  

대기 이벤트: transactions - statements - stages - waits  

MySQL이 너무 오래 기다리면 대기-오류 관계인 시간 초과(timeout)가 발생합니다.  
- max_execution_time
- lock_wait_timeout
- innodb_lock_wait_timeout
- connect_timeout
- wait_timeout

이 설정을 사용하되 의존해서는 안 됩니다.  

### 오류 메트릭

오류 메트릭은 오류를 나타냅니다.  
대기와 마찬가지로 오류도 비율로 계산되지만, MySQL이나 클라이언트가 실패한 시기를 나타내므로 별도로 분류할 가치가 있습니다.  

### 접근 패턴 메트릭

접근 패턴 메트릭은 애플리케이션이 어떻게 MySQL을 사용하는지를 나타냅니다.  
Com_select 접근 패턴 메트릭은 실행된 SELECT 문의 수를 계산합니다.  
응답 시간이 형편없고 Select_full_join 접근 패턴 메트릭이 높다면 이는 결정적인 증거입니다.  

### 내부 메트릭

MySQL 엔지니어나 사용자는 알 필요도 없고 관심을 둘 필요도 없기 때문입니다.  
그러나 이것은 이 분야에서 가장 흥미로운 부분이며, MySQL을 더 자세히 이해하고 싶을 때를 대비하여 충분히 알았으면 합니다.  


## 6-5 스펙트라

지금부터는 MySQL 메트릭을 스펙트라로 분류하여 살펴봅니다.  
각 스펙트럼을 구성하는 MySQL 메트릭과 시스템 변수에 관해 정확하게 이야하기려면 **메트릭 명명 규칙**이 필요합니다.  
MySQL에는 메트릭 명명 규칙이 없으며 업계 표준도 없기 때문입니다.  

- 전역: MySQL 서버 전체(모든 클라이언트, 모든 사용자, 모든 쿼리 등)
- 세션 메트릭: 단일 클라이언트 연결로 범위가 지정된 전역 메트릭
- 요약 메트릭: 계정, 호스트, 스레드 트랜잭션 등 다양한 측면으로 범위가 지정된 전역 메트릭의 하위 세트


### 쿼리 응답 시간

놀랍게도 MySQL 8.0 버전까지 이 메트릭이 없었습니다.  

```
SELECT
  ROUND(bucket_quantile * 100, 1) AS p,
  ROUND(BUCKET_TIMER_HIGH / 1000000000, 3) AS ms
FROM performance_schema.events_statements_histogram_global
WHERE bucket_quantile >= 0.95
ORDER BY bucket_quantile
LIMIT 1;
```
이 쿼리는 정확하지는 않지만 P95에 매우 가까운 백분위수를 반환합니다.  

#### MySQL 5.7과 이전 버전

MySQL 5.7과 이전 버전은 전역 쿼리 응답 시간 메트릭이 표시되지 않습니다.  
전역 응답 시간을 계산하려면 모든 쿼리에서 집계해야 합니다.  

#### Percona 서버 5.7

플러그인을 설치하는 것은 쉽지만 응답 시간 범위의 히스토그램이므로 구성하고 사용하려면 작업이 필요합니다.  

#### MariaDB 10.0

MariaDB는 Percona의 똑같은 플러그인을 사용하지만 '쿼리 응답 시간 플러그인'으로 이름이 약간 다릅니다.  


### 오류

쿼리를 실행하여 성능 스키마에서 모든 오류 수를 쉽게 얻을 수 있습니다.  
```
SELECT SUM(SUM_ERROR_RAISED) AS global_errors
FROM performance_schema.events_errors_summary_global_by_error
WHERE ERROR_NUMBER NOT IN (1287);
```

MySQL에는 오류와 경고가 너무 많아서 전역 오류율이 얼마나 될지 알 수 없습니다.  
불가능한 오류율 0을 기대하거나 달성하려고 하지 마세요.  
목표는 애플리케이션의 일반 오류율을 정립하는 것입니다.  

쿼리를 실행하여 성능 스키마에서 전체 쿼리의 오류 수를 얻을 수 있습니다.  
```
SELECT SUM(sum_errors) AS query_errors
FROM performance_schema.events_statements_summary_global_by_event_name
WHERE event_name LIKE 'statement/sql/%';
```

애플리케이션이 MySQL에 네트워크 연결을 만들 수 없을 때 MySQL은 클라이언트를 인식하지 못하고 연결 오류를 보고하지 않습니다.  
따라서 로-레벨의 네트워크 연결 문제는 애플리케이션에서 보고해야 합니다.  


### 쿼리

쿼리와 관련된 메트릭은 MySQL이 얼마나 빨리 작동하고 어떤 유형의 작업을 수행하고 있는지 매우 높은 수준에서 보여 줍니다.  

#### QPS

`QPS = (Queries T1 - Queries T0) / (T1 - T0)`  

QPS는 얼마나 빠르게 쿼리를 실행하는지와 같은 전반적인 MySQL 처리량을 나타내므로 많은 관심을 둡니다만, 이것에만 의존해서는 안 됩니다.  
QPS가 매우 높지만 응답 시간도 길다면 QPS는 성능이 급격히 떨어지고 문제가 있음을 나타냅니다.  

KPI라는 관점에서 QPS에 문제가 있다면 다른 메트릭을 활용해서 이 문제를 찾을 수 있습니다.  

`Questions`은 클라이언트가 보낸 쿼리만 계산합니다.  
트리거에 의해 실행된 쿼리는 클라이언트가 전송하지 않았으므로 `Questions`에는 포함되지 않지만 `Queries`에는 포함됩니다.  

#### TPS

명시적인 다중 명령문 트랜잭션이 중요한 애플리케이션이라면 초당 트랜잭션이 QPS만큼 중요합니다.  

- Com_begin
- Com_commit
- Com_rollback

일반적으로 `Com_begin`과 `Com_commit`의 비율은 같습니다.  
트랜잭션 지연 문제가 발생하면 `Com_begin` 비율이 다른 두 메트릭보다 높습니다.  

트랜잭션 처리량은 성공한 트랜잭션을 의미하므로 `Com_commit`을 사용하여 TPS를 측정합니다.  
트랜잭션 롤백은 오류를 나타내야 할 것 같지만, `ROLLBACK`문은 정리에도 사용 됩니다.  

#### 읽기/쓰기

접미사 `_multi`는 다중 테이블을 참조하는 쿼리를 나타냅니다.  

이러한 메트릭은 `Queries`를 완전히 설명하는 것이 아니라 성능과 관련하여 가장 중요한 메트릭일 뿐입니다.  
`Queries`는 `SHOW`, `FLUSH`, `GRANT` 등과 같은 다른 유형의 SQL 문을 설명하기 때문에 읽기와 쓰기 백분율은 100%가 아닙니다.  

#### 관리

관리 메트릭을 확인하려면 일반적으로 데이터베이스 관리자만 사용할 수 있는 명령을 이용합니다.  
`Com_admin_commands`로 문제를 확인할 수는 없지만, 이를 모니터링하는 것이 좋은 습관입니다.  

#### SHOW

MySQL에서 스레드, 시간과 리소스를 사용합니다.  


### 스레드와 연결

`Threads_running`은 활성 쿼리 실행과 직접적 연결되므로 MySQL이 얼마나 열심히 일하는지를 알려주며 CPU 코어 개수는 이를 효과적으로 제한합니다.  

스레드와 연결에 대한 가장 중요한 4가지 메트릭은 다음과 같습니다.  

- Connections
- Max_used_connections
- Threads_connected
- Threads_running

`Connections`은 성공과 실패를 포함한 MySQL에 대한 연결 시도 횟수입니다.  
이 메트릭이 비정상으로 높은 연결 속도를 나타내면 근본 원인을 찾아서 수정해야 합니다.  

`Max_used_connections`는 연결 사용률을 나타냅니다.  
각 애플리케이션 인스턴스에는 고유한 연결 풀이 있으므로 애플리케이션에 더 많은 연결이 필요합니다.  

애플리케이션이 성능을 높이거나 수천 명의 사용자를 지원하려면 수천 개의 MySQL 연결이 필요하다는 것은 잘못된 생각입니다.  
제한 요소는 연결이 아니라 스레드입니다.  
대부분의 애플리케이션에서는 수백 개의 연결이면 충분합니다.  
애플리케이션에 수천 개의 연결이 필요할 때는 샤딩해야 합니다.  

모니터링하고 피해야 할 진짜 문제는 100% 연결 사용률입니다.  
연결 사용률이 갑자기 증가하여 100%에 가까워지면 원인은 항상 외부 문제이거나 버그, 또는 둘 다입니다.  

클라이언트가 연결되거나 끊어지면 MySQL은 `Threads_connected` 게이지 메트릭을 늘리거나 줄입니다.  
MySQL이 클라이언트 연결당 하나의 스레드를 실행한다는 것을 반영합니다.  

`Threads_running`은 게이지 메트릭이며 CPU 코어 수와 관련된 암시적 사용률입니다.  
`Threads_running`이 수백, 수천으로 급증하더라도 성능은 CPU 코어 수의 약 두 배 수준인 훨씬 낮은 값에서 급격하게 떨어집니다.  
실행 중인 스레드 수가 CPU 코어 수보다 많으면 일부 스레드가 중단되어 CPU 시간을 기다리고 있음을 의미합니다.  
따라서 `Threads_running`이 30 미만으로 매우 낮아야 정상입니다.  


### 임시 개체

임시 개체는 MySQL이 행 정렬, 대규모 조인 등 다양한 목적으로 사용하는 임시 파일과 테이블입니다.  
임시 개체는 생성되는 비율이 안정적이라면 일반적이고 해롭지 않으므로 이러한 메트릭은 거의 0이 아닙니다.  

- Created_tmp_disk_tables
  - 가장 영향력 있는 메트릭
  - 임시 테이블을 디스크에 쓸 경우
- Created_tmp_tables
  - GROUP BY와 같이 임시 테이블이 필요한 경우
- Created_tmp_files


### 준비된 명령문

Prepared statements: 값을 바인딩 해서 사용
```
id := 75
SELECT * FROM tbl WHERE id = ?
```

`Com_stmt_execute`는 `Com_stmt_prepare`보다 훨씬 커야 합니다.  
가장 나쁜 경우는 두 메트릭이 1:1이거나 이에 가까울 때로, 한 번은 실행 준비, 한 번은 구문 종료 등 하나의 쿼리가 두 번 MySQL에 전달되기 때문입니다.  

성능 영향 외에도 애플리케이션이 의도하지 않게 준비된 명령문을 사용할 수 있으므로 이러한 준비된 명령문 메트릭을 모니터링해야 합니다.  

열린 준비된 명령문의 수는 `var.max_prepared_stmt_count`개로 제한되며 기본 값은 16,382입니다.  
`Prepared_stmt_count`가 `var.max_prepared_stmt_count`에 도달하지 않도록 하세요.  
이런 일이 발생하면 준비된 명령문 누수 때문에 애플리케이션 버그가 됩니다.  


### 잘못된 SELECT

4가지 메트릭은 일반적으로 성능에 좋지 않은 SELECT 문의 발생을 카운트합니다.  
- Select_scan
- Select_full_join
- Select_full_range_join
- Select_range_check

MySQL은 테이블을 조인할 때 전체 테이블 스캔 대신 인덱스를 사용하여 범위 스캔을 수행하므로 `Select_full_join`보다 `Select_full_range_join`이 낫습니다.  
`Select_range_check`는 `Select_full_range_join`과 비슷하지만 더 나쁩니다.  

`Select_full_join`과 `Select_range_check`는 0이 아니면 즉시 찾아서 수정해야 합니다.  


### 네트워크 처리량

일반적으로 MySQL이 네트워크에 영향을 주는 것이 아니라 오히려 네트워크가 MySQL에 영향을 줍니다.  

MySQL이 네트워크를 포화시키는 것을 딱 한 번 본 적이 있습니다.  
원인은 일반적으로 문제가 되지 않는 시스템 변수인 `var.binlog_row_image`와 관련이 있었습니다.  
애플리케이션에서 다음과 같은 요인이 동시에 발생하면 **퍼펙트 스톰**이 일어납니다.

- MySQL을 대기열로 사용
- 방대하 BLOB 값
- 쓰기 전용
- 높은 처리량

해결책은 복제할 필요가 없는 `BLOB`값 복제를 중지하기 위해 `var.binlog_row_image`를 `noblob`으로 변경하는 것이었습니다.  


### 복제

복제는 지연이라는 골칫거리를 만듭니다.  

MySQL에는 복제 지연에 대한 악명 높은 게이지 메트릭인 `Seconds_Behind_Source`가 있습니다.  
결과적으로 모범 사례는 이 메트릭을 무시하고 대신 pt-heartbeat와 같은 도구를 사용하여 실제 복제 지연을 측정하는 것입니다.  

MySQL은 복제와 관련된 악명 높지 않은 메트릭을 하나 제공합니다.  
`Binlog_cache_disk_use`입니다.  
바이너리 로그 캐시가 너무 작아 트랜잭션에 대한 모든 쓰기를 저장할 수 없을 때는 변경 사항이 디스크에 기록되고 `Binlog_cache_disk_use`가 증가합니다.  
빈도가 잦아지면 바이너리 로그 캐시 `var.bin_log_cache_size` 크기를 늘려서 완화할 수 있습니다.  


### 데이터 크기

보통은 데이터베이스가 예상보다 커지므로 데이터 크기를 모니터링하는 것이 중요합니다.  

MySQL은 정보 스키마 테이블(information_schema.tables)에서 테이블 크기를 제공합니다.  
데이터베이스와 테이블 크기 메트릭에 대한 표준은 없습니다.  
최소한 매시간마다 데이터베이스 크기를 수집하세요.  


### InnoDB

### 변경 내역 목록 길이(HLL)

HLL이 몇 분 또는 몇 시간 동안 크게 증가하면 InnoDB가 상당한 수의 이전 행 버전을 제거하지 않고 유지했음을 뜻하는데, 이는 하나 이상의 장 시간 실행 트랜잭션이 커밋되지 않았거나 알 수 없는 이유로 클라이언트 연결이 끊겨 롤백되지 않고 버려졌기 때문입니다.  

HLL이 100,000보다 크면 모니터링하고 경고해야 합니다.  
필요한 작업은 장시간 실행이나 버려진 트랜잭션을 찾아서 종료하는 것입니다.  

#### 교착 상태

교착 상태는 2개 이상의 트랜잭션이 다른 트랜잭션에 필요한 로우 락을 보유할 때 발생합니다.  
MySQL은 교착 상태를 해결하기 위해 트랜잭션 하나를 자동으로 감지하고 롤백하며 innodb.lock_deadlocks 메트릭 값을 늘립니다.  

고도로 동시성을 가지는 데이터 접근은 같은 행에 접근하는 서로 다른 트랜잭션이 대체로 같은 순서로 행을 검사하도록 애플리케이션을 설계하여 교착 상태를 방지해야 합니다.  

> https://helloworld.kurly.com/blog/vsms-performance-experiment/#%EA%B0%80%EC%9E%A5-%EA%B0%84%EB%8B%A8%ED%95%9C-%ED%95%B4%EA%B2%B0-%EB%B0%A9%EB%B2%95-%EC%A0%95%EB%A0%AC

#### 행 잠금

로우 락 메트릭은 잠금 경합을 나타냅니다.  
즉, 쿼리가 얼마나 빠르게 또는 느리게 데이터를 쓰기 위해 로우 락을 획득했는지를 나타냅니다.  

- innodb.lock_row_lock_time
  - 로우 락을 획득하는 데 소요된 전체 시간(ms)
  - T1 (500ms), T2 (700ms)인 경우 T2 - T1 = 200ms로 보고해야 함
- innodb.lock_row_lock_current_waits
  - 로우 락을 획득하기 위해 대기 중인 현재 쿼리 수에 대한 게이지 메트릭
- innodb.lock_row_lock_waits
  - 행을 획득하기 위해 대기한 쿼리 수
  - 대기 속도가 증가하다면 대기의 이유가 ㅣㅇㅆ다는 것이므로 문제가 있다는 의미
- innodb.lock_timeouts
  - 로우 락 대기 시간을 초과할 때 증가

#### 데이터 처리량

- innodb_data_read
- innodb_data_written

클라우드와 같이 스토리지 처리량이 제한적인 상황이라면 데이터 처리량 모니터링은 반드시 필요합니다.  
InnoDB는 매우 빠르고 효율적이지만 여전히 데이터와 디스크 사이의 복잡한 소프트웨어 계층으로 인해 게시된 스토리지 처리량 속도를 달성할 수 없습니다.  

스토리지가 로컬에 직접 연결되지 않았다면 처리량은 네트워크 속도로 제한 됩니다.  
1Gbps는 125MB/s와 같으며 이느 회전 디스크와 유사한 처리량입니다.  

#### IOPS

> IOPS (Input/Output Operation Per Second) 는 초당 처리되는 I/O 의 갯수라고 이해하면 된다.

InnoDB 성능의 존재 이유는 스토리지 I/O를 최적화하고 줄이는 데 있습니다.  
높은 IOPS는 엔지니어링 관점에서 인상적이지만 스토리지가 느리므로 성능의 골칫거리입니다.  

스토리지 IOPS의 최대치는 스토리지 장치에 의해 결정됩니다.  

백그라운드 작업을 위한 InnoDB I/O 용량은 주로 var.innodb_io_capacity와 var.innodb_io_capacity_max로 설정하며, 기본 값이 각각 200IOPS와 2,000IOPS인 시스템 변수입니다.  
백그라운드 작업은 페이지 플러싱, 변경 버퍼 병합 등을 포함합니다.  
백그라운드 작업 스토리지 I/O를 제한하면 InnoDB가 서버를 초과하는 일은 없습니다.  
반대로 포그라운드 작업에는 설정할 수 있는 I/O 용량이나 제한이 없으며 필요하고 사용 가능한 만큼의 IOPS를 사용합니다.  
주로 포그라운드 작업은 쿼리를 실행하는 것이지만 그렇다고 해서 쿼리가 IOP를 과도하게 사용한다는 의미는 아닙니다.  
읽기의 경우 버퍼 풀은 의도적으로 IOPS를 최적화하고 줄입니다.  
쓰기의 경우 페이지 플러싱 알고리즘과 트랜잭션 로그가 스토리지 I/O를 의도적으로 최적화하고 줄입니다.  

애플리케이션에는 높은 IOPS 달성을 방해하는 많은 계층이 있기 때문에 높은 IOPS를 달성하기 어렵다.  

InnoDB는 디스크가 아닌 메모리의 데이터로 작동합니다.  

#### 버퍼 풀 효율성

InnoDB 버퍼 풀은 테이블 데이터와 기타 내부 데이터 구조의 메모리 내 캐시입니다.  

접근할 때 데이터가 버퍼 풀에 없으면, InnoDB는 스토리지에서 데이터를 읽고 버퍼 풀에 저장합니다.  

- Innodb_buffer_pool_read_request
    - 버퍼 풀의 데이터에 접근하기 위한 모든 요청을 계산
- Innodb_buffer_pool_reads
    - 데이터가 메모리에 없으면 증가

버퍼 풀 효율성은 MySQL이 시작할 때는 매우 낮습니다.  
이것은 정상이며 **콜드 버퍼 풀**이라고 합니다.  

버퍼 풀 효율성은 100%에 매우 근접해야 하지만 값에 집착하지는 마세요.  

과거에는 캐시 적중률을 성능으로 생각했지만, 더는 사실이 아니며 지금은 쿼리 응답 시간이 곧 성능입니다.  

성능 병목 현상이 있으면 CPU나 스토리지에서 발생합니다.  

버퍼 풀 효율성은 이러한 세 가지 영향의 결합된 효과를 나타냅니다.  
- 데이터 접근: 데이터를 버퍼 풀로 가져옵니다.
- 페이지 플러싱: 버퍼 풀에서 데이터를 제거할 수 있습니다.
- 사용 가능한 메모리: 메모리에 보관하는 데이터가 많을수록 데이터를 가져오거나 플러시할 필요가 줄어듭니다.

그 값이 정상보다 낮으면 원인은 하나, 둘 또는 세 가지 모두일 수 있습니다.  
세 가지를 모두 분석하여 가장 크거나 변경하기 쉬운 것을 결정해야 합니다.  

#### 페이지 플러싱

- 프리 페이지: 데이터가 없고, 새 데이터를 불러올 수 있음
- 데이터 페이지: 수정되지 않은 데이터 포함
- 더티 페이지: 디스크로 플러시되지 않은 수정된 데이터 포함
- 기타 페이지: 기타 내부 데이터

- innodb.buffer_pool_pages_total
    - 버퍼 풀 크기에 따라 달라지는 버퍼 풀의 총 페이지 수
    - innodb.buffer_pool_pages_free와 innodb.buffer_pool_pages_dirty를 각각 총 페이지로 나눈 값

프리 페이지가 지속해서 프리 페이지 대상보다 훨씬 높거나 줄어들지 않으면 버퍼 풀이 너무 큰 것입니다.  

데이터 쓰기는 더티 페이지가 발생하므로 더티 페이지의 급증은 IOPS와 트랜잭션 로그 메트릭의 급증을 뒷받침합니다.  
결국 더티 페이지는 페이지 플러싱에 따라 늘었다 줄었다를 반복합니다.  

페이지 플러싱은 데이터 수정 사항을 디스크에 기록하여 더티 페이지를 정리합니다.  
페이지 플러싱은 지속성, 체크포인트, 페이지 제거라는 3가지가 밀접하게 관련된 목적을 수행합니다.  

페이지의 생명 주기  
프리 페이지 (데이터 적재)> 클린 페이지 (데이터 수정)> 더티 페이지 (수정 사항 플러시)> 클린 페이지 (버퍼 풀에서 제거)> 프리 페이지

LRU 목록은 데이터가 있는 모든 페이지를 추적하며, 더티 페이지를 포함합니다.  
플러시 목록은 더티 페이지만 명시적으로 추적합니다.  

**적응형 플러싱**은 페이지 클리너가 플러시 목록에서 더티 페이지를 플러시하는 속도를 결정합니다.  
이 알고리즘은 트랜잭션 로그 쓰기 속도에 따라 페이지 플러시 속도를 변경하므로 적응형입니다.  

적응형 플러싱의 목적은 체크포인트로 트랜잭션 로그의 공간을 회수하는 것입니다.  

LRU 플러싱은 버퍼 풀에서 오래된 페이지를 플러시하고 제거합니다.  
LRU 플러싱은 백그라운드와 포그라운드에서 발생합니다.  
이때 대기가 발생하므로 쿼리 응답 시간이 늘어나 성능에 좋지 않습니다.  

페이지 클리너는 백그라운드 LRU 플러싱을 처리합니다.  

유휴 플러싱은 InnoDB가 쓰기를 처리하지 않을 때 발생합니다.  

- innodb.buffer_flush_batch_total_pages: 모든 알고리즘에 대한 총 페이지 플러시 속도
- innodb.buffer_flush_adaptive_total_pages: 적응형 플러싱에 의해 플러시된 페이지 수
- innodb.buffer_LRU_batch_flush_total_pages: 백그라운드 LRU 플러싱에 의해 플러시된 페이지 수

플러싱 알고리즘마다 속도가 다르지만 플러싱에는 IOPS가 소요되므로 스토리지 시스템이 모든 알고리즘의 기반이 됩니다.  
IOPS에는 특히 클라우드에서 마이크로초에서 밀리초 범위의 대기 시간이 있음을 기억해야 합니다.  


#### 트랜잭션 로그

트랜잭션 로그는 지속성을 보장합니다.  
트랜잭션이 커밋되면 모든 데이터 변경 사항이 트랜잭션 로그에 기록되고 디스크에 플러시되므로 데이터 변경 사항이 지속 가능해지며 해당 더티 페이지는 메모리에 남게 됩니다.  
트랜잭션 로그 플러싱은 페이지 플러싱이 아닙니다.  
두 프로세스는 분리되어 있지만 서로 뗄 수 없는 관계입니다.  

트랜잭션 로그에는 페이지가 아닌 데이터 변경 사항이 포함되지만, 데이터 변경 사항은 버퍼 풀의 더티 페이지에 연결됩니다.  
트랜잭션이 커밋되면 데이터 변경 사항이 트랜잭션 로그의 헤드에 기록되고 디스크에 플러시되어 헤드가 시계 방향으로 진행됩니다.  

체크포인트 수명은 헤드와 테일 사이의 트랜잭션 로그 길이입니다.  
더티 페이지가 플러시되면 트랜잭션 로그의 해당 데이터 변경 사항을 새 데이터 변경 사항으로 덮어쓸 수 있습니다.  
체크포인트 수명은 너무 오래되지 않도록 테일을 전진시킵니다.  

우리에게 의미 있고 모니터링해야 하는 중요한 지표는 체크포인트 수명이 **트랜잭션 로그 사용률**과 비동기 플러시 지점에 얼마나 가까운가입니다.  
트랜잭션 로그 사용률은 비동기 플러시 지점이 로그 파일 크기의 6/8이기 때문에 보수적입니다.  
따라서 트랜잭션 로그 사용률이 100%일 때 로그의 25%는 새 쓰기를 기록할 수 있지만, 서버 성능은 비동기 플러시 지점에서 눈에 띄게 떨어진다는 점을 기억해야 합니다.  

`innodb.log_waits`는 0이어야 합니다.  
그렇지 않으면 로그 버퍼 크기는 `var.innodb_log_buffer_size`에 의해 구성됩니다.  
트랜잭션 로그는 디스크에 있는 2개의 물리적 파일로 구성되므로 데이터 변경 사항을 디스크에 기록하고 동기화하는 것이 가장 기본 작업입니다.  

다음 2개의 게이지 메트릭은 얼마나 많은 작업이 대기 중인지 보고합니다.  
- innodb.os_log_pending_writes
- innodb.os_log_pending_fsyncs

쓰기와 동기화는 매우 빠르게 발생해야 하므로 이러한 메트릭은 항상 0이어야 합니다.  

로그 파일 크기를 결정하기 위한 기준으로 시간당 기록된 총 로그 바이트를 모니터링하는 것이 가장 좋습니다.  


## 6-6 모니터링과 경보

### 레졸루션

레졸루션은 메트릭이 수집되고 보고되는 빈도를 의미합니다.  

최소한 5초 이상의 레졸루션으로 KPI를 수집해야 합니다.  
로깅되는 쿼리 메트릭과 달리, MySQL 메트릭은 수집되거나 또는 영구히 사라지므로 될 수 있는 한 가장 높은 레졸루션으로 맞추기 위해 노력해야 합니다.  

### 헛된 노력(임곗값)

임곗값이란 대기 중인 관리자를 호출하곤 하는 모니터링 경보 발동의 기준이 되는 값입니다.  
문제는 임곗값에도 지속 시간이 필요하다는 것입니다.  

임곗값은 완벽하기 어려운 것으로 악명이 높습니다.  
여기서 완벽함이란 진짜 문제가 될 만한 것에만 경고하고 잘못 판단하지 않는 것을 의미합니다.  

### 사용자 경험과 객관적 한계에 대한 경고

임곗값 대신 작동하는 2가지 검증된 솔루션이 있습니다.  

- 사용자 경험에 대한 경고
- 객관적 한계에 대한 경고

사용자가 경험하는 MySQL 메트릭은 응답 시간과 오류 2가지뿐입니다.  
이는 사용자가 경험하기 때문만이 아니라 잘못 측정할 수 없으므로 신뢰할 수 있는 신호입니다.  

전체 20초 동안 오류가 초당 10개로 증가하면 사용자 경험이 나빠지나요?  
그렇다면 이를 임곗값과 지속 시간으로 만드세요.  

사용자가 애플리케이션의 1초 미만 응답에 익숙하다면 1초 응답은 현저히 느리게 느껴집니다.  

다음은 MySQL 외부의 일반적인 객관적 한계입니다.  
- 사용 가능한 디스크 공간 없음
- 사용 가능한 메모리 없음
- 100% CPU 사용률
- 100% 스토리지 IOPS 사용률
- 100% 네트워크 사용률

MySQL에는 `AUTO_INCREMENT` 열이 최댓값에 근접하고 있는지 확인하는 기본 메트릭이나 방법이 없습니다.  


### 원인과 결과

MySQL 응답 속도가 느리다면 애플리케이션이 원인일 때가 대부분입니다.  
애플리케이션이 MySQL을 구동하기 때문입니다.  

실제로는 원인은 모니터링과 로깅이 허용하는 만큼만 알 수 있습니다.  
문제가 발생하기 전에는 모든 것이 정상이었기 때문에 MySQL을 구동하는 애플리케이션부터 정상으로 돌아가는 것이 목표입니다.  


## 요점 정리

- MySQL KPI는 응답 시간, 오류, QPS, 실행 중인 스레드입니다.
- 메트릭 클래스는 서로 관련되어 있습니다. 속도는 사용율을 높이고, 사용율은 다시 속도를 감소시키며 높은 사용률은 대기를 발생시키고, 대기 시간이 초과되면 오류가 발생합니다.
- MySQL 서버 성능은 MySQL을 통한 워크로드의 굴절인 메트릭 스펙트럼을 통해 드러납니다.




