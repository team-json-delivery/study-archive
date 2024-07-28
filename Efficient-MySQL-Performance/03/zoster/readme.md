# Chapter 3.Data
* 여기서는 Indirect query optimization을 시작한다
* 직접 쿼리 최적화를 잘 해왔고 인덱스가 잘 구성되어 있어도 여전히 느릴 수 있다
* 이때 쿼리가 접근하는 데이터부터 시작해 쿼리를 중심으로 최적화를 시작해야 한다
* TRUNCATE TABLE은 성능을 크게 향상시키지만 이것을 최적화로 사용할 수는 없다
* 논리적 결과에 따른 위험성으로 잘 사용하진 않지만 적은 데이터가 더 나은 성능을 가져오는 것을 증명한다
## 3-1 Three Secret
### Indexes May Not Help
* 인덱스는 성능에서 핵심이지만 좋은 인덱스라도 쿼리가 느릴 수 있다
* 인덱스와 인덱싱을 잘 알고나면 인덱스 스캔과 테이블 스캔을 피하는 데 너무 능숙해져서 인덱스 조회만 남게 되는데 여전히 아이러니한 문제가 있다
* 인덱스 없이는 성능을 달성할 수 없지만 인덱스가 silver bullet인 것은 아니다
* 더 이상 쿼리와 인덱스 최적화가 불가능하다면 다음 단계는 간접 쿼리 최적화이다
#### Index scan
* 인덱스 스캔은 테이블이 커질수록 인덱스도 함께 커지므로 영향력이 점점 감소한다
* 인덱스 스캔은 테이블 행 수가 증가할수록 인덱스 스캔을 사용하는 쿼리에 대한 응답 시간도 늘어나므로 반드시 지연이 발생한다
#### Finding rows
* 인덱스 조회를 사용하는 느린 쿼리를 최적화할 때 확인하는 첫 번째 쿼리 메트릭은 조회된 행이다
* 좋은 인덱스를 사용하더라도 쿼리가 너무 많은 행을 검사할 수도 있다
```python
☐ system
☐ const
☐ eq_ref
☐ unique_subquery
```
> 한 행만 일치하는 인덱스 조회 접근 유형
* EXPLAIN 계획의 type 필드가 위 나열한 접근 유형 가운데 하나가 아니라면 rows 필드와 조회된 쿼리 메트릭 행에 주의를 기울여야 한다
#### Joining tables
* 테이블을 조인할 때 각 테이블의 몇 개 행이 성능을 빠르게 떨어뜨릴 수 있다
* 테이블당 100개 행만 있는 3개 테이블 조인은 100만개 행을 접근한다
  * 이를 막기 위해 위에 나열된 접근 유형 중 하나를 사용하여 하나의 행만 일치하는 것이 가장 좋다
* 잘못된 조인에 대한 해결책은 다른 테이블에서 더 나은 인덱스를 생성하여 MySQL이 조인 순서를 변경하는 것이다
#### Working set size
* 인덱스는 메모리에 있을 때만 유요하다, 쿼리가 조회하는 인덱스값이 메모리에 없으면 MySQL은 디스크에서 값을 읽는다
* 여기서 문제는 인덱스가 메모리를 놓고 경쟁한다는 것이다.
* DBA는 보통 전체 데이터 크기의 10%에 해당하는 메모리를 할당한다
* 작업 세트의 크기가 사용할 수 있는 메모리보다 훨씬 커지면 인덱스가 도움이 되지 않을 수 있다
* 메모리를 키워서 해결할 수 있지만 스케일업은 지속가능한 방법이 아니며 근본적 해결(샤딩)과 같은 방법을 사용해야 한다
### Less Data Is Better
* 데이터의 크기가 적절한 것이 큰 것보다는 당연히 더 쉽다
* 데이터를 무의식적으로 다 저장하지 않는 것은 훌륭하지만, 동료들은 데이터를 제한하는 엔지니어에게 불만이 있을 수 있다
> 로그의 경우에도 같은 반응
* 그렇다 하더라도 데이터 크기 때문에 문제가 발생하기 전에 제한 업이 증가하는 데이터에 브레이크를 걸어야 한다
### Less QPS Is Better
#### QPS is only a number - a measurement of raw throughput
* QPS는 일반적으로 쿼리나 성능에 대한 질적인 정보를 제공하지 않는다
#### QPS values have no obhective meaning
* QPS는 좋거나 나쁘지 않고, 높거나 낮지도 않고, 전형적이거나 비전형적이지도 않다
* QPS는 또한 시간, 요일 계절, 휴일에 따라서도 달라 질 수 있다
#### It is difficult to increase QPS
* QPS는 높일 수 없다
---
* QPS는 business 가치를 의미하지 않고 애플리케이션에만 관련이 있다
* QPS는 자산이라기 보다는 부채에 가깝다 따라서 QPS가 낮을수록 좋다
## 3-2 Priciple of Least Data
* 최소 데이터 원칙은 `필요 데이터만 저장과 접근`으로 정의한다
### Data Access
* 필요 이상으로 많은 데이터에 접근해선 안된다
* 접근은 모든 MySQL의 작업을 의미한다
```python
☐ Return only needed columns
☐ Reduce query complexity
☐ Limit row access
☐ Limit the result set
☐ Avoid sorting rows
```
> 효율적인 데이터 접근 점검표
* 데이터 접근이 효율적이라면 MySQL은 거의 in-memory 캐시만큼 빠르게 동작할 수 있다
* 더는 쿼리를 최적화할 수 없고 데이터 크기를 줄일 수 없을 때는 접근 패턴을 변경해서 쿼리를 최적화해야 한다
#### Return only needed columns
* 쿼리는 필요한 열만 반환해야 한다, 특히 BLOB,TEXT,JSON 열이 있다면 더욱 그렇다(select * 을 사용하지 말자)
> 애초에 필요 없는 열이 저장이 안되어야 하지 않을까?
#### Reduce query complexity
* 쿼리는 될 수 있는 한 단순해야 한다
* 복잡한 쿼리는 MySQL 보다는 엔지니어에게 문제가 된다, 분석하고 최적화가 어렵기 때문이다
* 처음부터 단순하게 짜고, 여력이 있다면 쿼리 복잡성을 줄여야 한다
#### Limit row access
* 쿼리는 될 수 있는 한 적은 수의 행에 접근해야 한다
* 의도하지 않아도 시간 경과에 따른 데이터 증가가 일반적인 원인이 된다
* 또는 단순히 실수로 인해 그렇게 될 수 있다
* 여기서 가장 중요한 원인은 범위와 목록을 제한하지 않는 것이다, ORDER BY LIMIT을 사용하여 미리 방어해두자
#### Limit the result set
* 쿼리는 될 수 있는 한 적은 수의 행을 반환해야 한다
* 첫 번째 변형은 애플리케이션이 일부 행을 사용할 때 발생하지만 모두 그렇지는 않다
  * 더 나은 WHERE 조건을 사용하는 대신 행을 필터링하는 애플리케이션에서 확인할 수 있다
  * > NOSQL 적 사고
* 두 번째 변형은 쿼리에 ORDER BY 절이 있고 애플리케이션이 정렬된 행중 일부를 사용할 때 발생한다
  * 필요한 만큼만 반환하자, top 20을 쓸거라면 LIMIT 20을 추가하자
* 세 번째 변형은 애플리케이션이 결과 세트를 집계하기만 할 때 발생한다
#### Avoid soring rows
* 쿼리는 행 정렬을 피해야 한다
* LIMIT 이 없는 ORDER BY 절은 삭제할 수 있고 애플리케이션이 대행할 수 있다는 신호다
### Data Storage
* 필요 이상으로 많은 데이터를 저장하지 말자
* 데이터는 우리에게 중요하지만 MySQL에는 부담이 된다
* 데이터 스토리지를 감사하면 놀라운 부분을 쉽게 볼수 있어 권장한다
```python
☐ Only needed rows are stored
☐ Every column is used
☐ Every column is compact and practical
☐ Every value is compact and practical
☐ Every secondary index is used and not a duplicate
☐ Only needed rows are kept
```
#### Only needed rows are stored
* 애플리케이션의 변경사항이 늘어남에 따라 엔지니어가 추적을 못하게 될 수도 있다
* 무엇을 저장하는지 검토한 지 오래되었거나, 신규라면 저장되는 데이터를 살펴보자
#### Every column is used
* 한 단계 더 들어가서 필요한 행만 저장하는 것 보다 필요한 열만 저장하는 것을 보자
* MySQL은 사용된 DB, table, index를 추적할 수 있지만 열은 추적할 수 없다
#### Every column is compact and practical
* 여기서 또 한 단계 더 들어가서 모든 열을 간결하고 실용적으로 만들어야 한다
* VARCHAR(255), BLOB, TEXT, JSON과 같은 형태는 매우 보수적으로 사용하자
#### Every value is compact and practical
* 필요한 행만 저장하는 것보다 세 단계 더 들어가면 모든 값을 간결하고 실용적으로 만들어야 한다
```roomsql
SELECT
 /*!40001 SQL_NO_CACHE */
 col1,
 col2
FROM
 tbl1
WHERE
 /* comment 1 */
 foo = ' bar '
ORDER BY col1
LIMIT 1; — comment 2
```
```roomsql
SELECT /*!40001 SQL_NO_CACHE */ col1, col2 FROM tbl1 WHERE foo=' bar ' LIMIT 1
```
* Query의 크기 자체를 축소하고 값을 최적화 하는 방식이다
* 여기서 쿼리의 결과가 달라지지 않도록 주의해야 한다
* Encoding에서도 이러한 형태로 IP를 숫자로 저장하거나, UUID를 binary로 저장하는 방식을 사용하여 데이터 크기를 줄일 수 있다
* 정규화를 통해서도 데이터를 줄일 수 있는데
* 속도를 위해 비정규화를 섣부르게 수행하지 말아라
#### Every secondary index is used and not a duplicate
* 모든 seondary index가 사용되어야 하며, 중복되지 않아야 한다
* 자주 사용되지 않는 인덱스 찾기는 까다롭기 때문에 삭제시 주의해야 한다
#### Only needed rows are kept
* 필요한 행만 유지해야 한다
* 저장할 때 행이 필요했지만 시간이 지남에 따라 필요없어 졌을 수 있다
* 삭제하자

## 3-3 Delete or Archive Data
* 데이터가 관리하기 어려울 정도로 쌓이는데 지켜보기만 하면 문제가 발생한다
* 문제가 인식되었을 때는 처리가 이미 어려울 수 있다
* 데이터를 archive 하려면 먼저 복사한 다음 지워야 하는데
  * 데이터 복사는 영향이 없도록 nonlocking SELECT 문을 사용하여 복사해야 한다
### Tools
* 도구를 만들어서 지우자
* 반드시 throttling을 위한 방법을 마련해야 한다
### Batch Size
* 일반적으로 단일 DELETE 문에서 1,000개 이하를 삭제하는 것이 안전하다
* 배치 크기는 실행 시간으로 조정하며 500ms는 좋은 시작점이다
#### Replication lag
* 7장에 자세히 나옴
#### Throttling
* 데이터 삭제 시 스로틀링 조절이 가장 중요하다
* 복제 지연을 모니터링하고 여기에 맞는 값을 세팅해야 한다
* 시간대에 따라 달라질 수 있으므로 보수적으로 잡자 
* 최대 traffic을 기준으로 작성하거나, 한가한 시간을 이용하여 작업하자
### Row Lock Contention
* 쓰기 작업이 많은 워크로드는 대량 작업으로 인해 row lock contention이 발생할 수 있다
### Space and Time
* 데이터를 삭제해도 디스크 공간이 확보되지 않는다(linux와 동일)
* 다른 애플리케이션과 물리적 공간을 공유한다면 다른 애플리케이션의 공간을 낭비해선 안된다
* 회수하는 가장 좋은 방법은 no operation인 `ALTER TABLE...ENGING=INNODB` 문을 실행하여 재구성 하면 된다
### The Binary Log Paradox
* 데이터를 삭제하면 데이터가 생성된다
* 데이터의 변경이 바이너리 로그에 기록되기 때문이다
* 바이너리 로깅은 비활성화 할 수 있지만 복제본은 바이너리 로그로 복제되기 때문에 PROD에선 불가능하다고 봐야한다
* full, minimal, noblob 세 가지 설정이 있는데
* 전체 행에 의존하는 외부 서비스가 없는 경우 minimal, noblob이 권장된다

## SUMMARY
* Less data yields better performance.
* Less QPS is better because it’s a liability, not an asset.
* Indexes are necessary for maximum MySQL performance, but there are caseswhen indexes may not help.
* The principle of least data means: store and access only needed data.
* Ensure that queries access as few rows as possible.
* Do not store more data than needed: data is valuable to you, but it’s dead weightto MySQL.
* Deleting or archiving data is important and improves performance.