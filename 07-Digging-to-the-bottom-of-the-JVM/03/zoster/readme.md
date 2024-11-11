# Chapter 3. GC & Memory allocation strategy
## 3.1 Introduction
* 다양한 Memory overflow와 leak 문제를 해결해야 하는 상황 또는 high consistency를 달성하는데 GC가 방해가 되는 상황이 오면 적절히 모니터링하고 조율하기 위해 알아야 한다
* Native method stack은 결정적이지만 Java heap / method 영역은 불확실성이 크다, 런타임에만 알 수 있기 때문이다

## 3.2 대상이 죽었는가?
### 3.2.1 Reference counting algorithm
* Java VM에서는 Reference counting algorithm을 사용하지 않는다, circular reference 문제를 풀기 어렵기 때문이다

### 3.2.2 Reachability analysis algorithm
* 주류 프로그래밍 언어들은 생사 판단에 reachability analysis algorithm을 이용한다
* 시작 노드에서 출발하여 탐색해 들어가고 그 과정에서 만들어지는 경로를 reference chain 이라고 한다
* 어떤 객체와 GC root 사이를 이어주는 reference chain이 없다면 GC root로 부터 그 객체에 도달할 방법이 없다는게 확실해진다
* GC root에 사용될 수 있는 객체는
1. VM stack
2. Method영역에서 클래스가 static field로 참조하는 객체
3. method영역에서 constant로 참조되는 객체
4. JVM 내부에서 쓰이는 참조(NPE, OOM..)
5. Synchronized keyword로 잠겨있는 객체
6. JMXBean
* 현재 최신 GC 들은 모두 부분 컬렉션을 지원한다. 또한 GC 루트가 너무 많아지지 않도록 최적화를 적용한다

### 3.2.3. 다시 참조 이야기로
* ```
  참조 타입 데이터에 저장된 값이 다른 메모리 조각의 시작 주소를 뜻한다면, 이 참조 데이터를 해당 메모리 조각이나 객체를 참조한다고 말한다
  ```
* 이 정의는 `버리기는 아까운` 객체를 표현할 방버이 없다
* JDK 1.2 부터 참조 개념이 확장되어 네 가지로 구분하기 시작했다
1. Strong reference
2. Soft reference
3. Weak reference 
4. Phantom referece
* 각 단계에 따라 GC가 회수하는 시점이 달라진다

### 3.2.4 살았나 죽었나?
* 도달 불가능하다고 바로 죽이는건 아니다, `유예` 단계가 남아있다.
* GC root에 reference chain이 없으면 먼저 첫번째 표시가 이루어진다. 여기서 finalize의 여부를 검사한다
* Finalize가 필요하다면 대기열에 추가된다, 그런데 finalize는 실행까지만 담보되고 완료는 담보되지 않는다.
  * 따라서 Finalize에 중요한 동작을 두는 것은 위험하다
  * 또한 실행하는 비용도 높고 순서도 보장되지 않는다.

### 3.2.5 Reclamation method area
* GC가 반드시 method 영역을 회수해야 하는 것은 아니다.
* `비용 효율`이 떨어지기 때문이다.

## 3.3 GC algorithm
* GC 구현에는 많은 기법이 활용되며 VM 또는 플랫폼에 따라 차이가 많다
* 객체의 생사여부를 기준으로 `reference counting algorithm` 과 `tracing GC`로 나눌 수 있다

### 3.3.31 세대 단위 컬렉션 이론
* 현재 대부분의 GC는 세대 단위 컬렉션 이론에 기초해 설계되었다
* 기본적으로 두 가지 가설이 뿌리를 이루는데
1. weak generational hypothesis : 대다수 객체는 일찍 죽는다
2. strong generational hypothesis : GC 과정에서 살아남은 횟수가 늘어날 수록 더 오래 살 가능성이 커진다
* 영역 안의 객체 대부분이 죽는다면, 한데 몰아넣고 살아남은 소수의 객체를 유지하는 것이 하나씩 표기하는 것보다 유리하다
* 그런데 단순히 세대를 나누기만 하면 문제가 발생한다, 신 세대를 연결한 구세대를 모두 찾아야 하는 문제
3. intergenerational reference hypothesis : 세대 간 참조의 개수는 같은 세대 안에서의 참조보다 훨씬 적다
* 이를 해결하기 위해 신세대에 Remembered set이라는 전역 데이터 구조를 하나 두고, 세대 간 참조가 있는지 기록해 관리하는 것이다

### 3.3.2 Mark-sweep algorithm
* 존 맥카시가 1960년에 제안한 방법
* 회수할 객체 모두에 표시하고 표시된 객체를 sweep 한다
* 단점은, 실행 효율이 일정하지 않다는 것과 메모리 파편화가 심하다는 것이 있다

### 3.3.3 mark-copy algorithm
* 회수할 객체가 많아질수록 효율이 떨어지는 mark-seep의 문제를 해결하기 위해 1969년 로버트 페니첼이 제안한 방법
* 가용 메모리를 절반으로 나누어 한쪽만 사용한다, 한쪽이 꽉 차면 살아남은 객체를 반대쪽에 복사하고 기존 블록을 모두 비운다
* 파편화로 부터 해방되지만 가용 메모리를 절반밖에 쓸 수 없다는 문제가 있다
* 1989년 앤드류 아펠은 이 특성을 반영해 `아펠 스타일 컬렉션`을 만들었다
* 신세대를 하나의 큰 eden, 두개의 작은 survivor로 나누고 하나의 survivor와 eden만 사용한다. GC 이후 살아남은 객체를 나머지 survivor로 옮기고 기존 공간을 지운다
* 8:1:1로 나누어 낭비하는 공간을 10%로 줄인다

### 3.3.4 mark-compact algorithm
* mark-copy는 객체 생존율이 높아질수록 효율이 떨어진다.
* 그래서 구세대에는 적합하지 않다 - `hypothesis 2`
* 1974년 에드워드 루더스가 제안한 방법으로 생존한 모든 객체를 메모리 영역의 한쪽으로 모으고 나머지 공간을 한번에 비운다
* 이로 인해 `stop the world` 가 발생하지만, 이동시키지 않는다면 결국 더 복잡한 할당을 할 수 밖에 없다

## 3.4 Hotspot algorithm 상세 구현
### 3.4.1 root node enumeration
* reachability analysis에서 reference chain을 찾는것을 말한다
* root node enumeration은 반드시 일관성이 보장되는 snapshot 상태에서 수행해야 한다
* 이것이 GC 시 모든 사용자 스레드가 일시 정지해야 하는 이유다(+Full GC)

### 3.4.2 Safe point
* Hotspot은 OopMap을 이용하여 GC root를 빠르고 정확하게 열거한다
* 그러나 모두 OopMap을 만들어 넣으면 메모리 효율이 떨어지기 때문에 safe point라고 하는 특정한 위치에만 기록한다
  * 일반적으로 method 호출, loop의 끝, 명시적인 예외 처리 지점이 된다

### 3.4.3 Safe region
* Safe point를 사용하면 모두 멈춰 세우고 GC를 수행하는 문제가 해결된 듯 보이지만 그렇지 않다
* 실행중이 아닌 thread(block, sleep 상태)는 safe point를 확인할 수 없기 때문
* safe region에 속하면 safe point와 관계없이 안전하다

### 3.4.4 Remembered set & card table
* 세대 간의 컬렉션 reference 문제를 해결하기 위해 존재
* 비회수 영역에서 회수 영역을 가리키는 포인터들을 기록한다
* 메모리를 효율적으로 사용하기 위해 적절한 정밀도를 선택하여 구현할 수 있겠다
* word precision, object precision, card precision
  * 이중 card precision으로 구현한 remembered set을 card table이라고 한다

### 3.4.5 Write barrier
* dirty field를 언제 set 할지는 명확하지만 어떻게 표시를 하느냐의 문제가 남아 있다
* `참조 타입 필드 대입` 시점에 끼어드는 AOP라고 할 수 있다. 대입되는 순간 around advice가 생성되어 대입 전후 추가 동작을 수행할 수 있게 하는 것이다
* 이는 오버헤드가 발생하지만 minor GC에서 old gen을 모두 스캔하는 비용보다는 저렴하다

### 3.4.6 동시 접근 가능성 분석
* 현재 주류 GC들은 모두 `reachablity analysis algorithm`을 사용하여 객체의 생사를 판단한다
* 이 과정에서 사용자 스레드는 모두 멈춰 있어야 한다
* 그러나 root node enumeration에서 gc root는 매우 갯수가 작고, OopMap과 같은 최적화로 인해 매우 짧고 상대적으로 일정하다
* 이후 gc는 객체 그래프를 탐색하는데 이 단계는 java의 heap 크기에 비례한다
* `tri-color marking` 기법은 이 과정을 줄이기 위한 방법이다
  * 흰색(방문한 적 없음), 검은색(방문, 스캔 완료), 회색(방문, 스캔 미완료)
  * GC root를 회색으로 바꾸고 자기 자신은 검은색으로 변환한다
  * 회색 객체를 탐색하고 도달한 객체를 검은색으로 변경
  * 탐색 이후 white인 객체는 모두 수거하여 GC

## 3.5 Classic GC
### 3.5.1 Serial collector
* 단일 스레드로 동작하는 Collector, 하나의 thread가 gc를 수행할 뿐 아니라, gc가 시작되면 다른 모든 스레드가 멈춰 있어야 한다
* single core라면 요구하는 메모리가 적고, 회수 효율이 높다는 장점이 있다
### 3.5.2 Parnew collector
* Serial의 YoungGen을 멀티 스레드로 확장한 버전, Old gen은 여전히 single thread
* YoungGen에는 mark-copy, OldGen에는 mark-compact를 이용한다
### 3.5.3 Parallel scavange collector
* 처리량 극대화를 위해 설계, YoungGen에 대해서 multithread로 동작한다
* mark-sweep-compact를 parallel하게 수행한다
### 3.5.4 Serial old collector
* Old gen에 대해서 serial 하게 처리하는 collector, serial collector의 old gen용 버전
### 3.5.5 Parallel old collector
* Parallel scavenge의 old gen용 버전
* Parallel scavange와 함께 조합하여 사용할 수 있다
### 3.5.6 CMS(Concurrent Mark-Sweep)
* `stop-the-world`를 최소화 하기 위한 collector
* web service의 경우 시스템의 정지 시간이 짧아야 사용자에게 더 나은 경험을 줄 수 있기 때문
* 단점은 프로세서의 자원을 많이 사용하여 자원에 민감하다
* 메모리 파편화가 발생할 가능성이 높다. 이 것이 반복되면 결국 full gc를 시도해야 한다
### 3.5.7 G1 collector(Garbage first collector)
* 힙 메모리를 Region 단위로 나누고, 필요에 따라 동적으로 GC를 수행
* Young Generation과 Old Generation을 하나의 Region으로 통합 관리
* 대규모 힙에서도 효율적이다
* CMS를 대체했으며 (jdk 9)
* 이를 위해 heap 전체가 아닌 region별 집합을 만들어 회수 영역을 관리하게 된다
* G1이 사용화되기 위해 해결해야 했던 주된 문제
  1. Java heap을 독립된 region으로 나누려면 객체간의 region간 참조 문제를 해결해야 한다
     2. remembered set을 이용하여 scan하는 방법이 있다, 이 구현을 위해 G1은 heap의 10~20%를 추가로 사용해야 한다
  3. 동시 표시 단계 동안 GC thread와 사용자 thread 간의 간섭 문제
     4. 각 region을 위해 TAMS라는 두 개의 포인터를 설계했다. region의 일부가 회수되는 동안 새로운 객체를 할당하기 위한 공간을 만들고 새로운 객체는 반드시 이 포인터보다 더 높은 주소 영역에 할당 해 mark 됨을 방지한다 
     4. CMS의 `동시 모드 실패` 처럼 full gc를 유발할 수 있다. 회수가 할당을 따라가지 못하는 경우 결국 full gc에 도달하게 된다
  5. 신뢰할 수 있는 정지 시간 예측 모델을 구현해야 한다

### 3.5.8 오늘날의 GC
* Serial Collector가 serial old 흡수, Parallel scavange와 Parallel old가 합쳐져 Parallel, CMS는 G1으로 대체
* ZGC / Shenandoah, 지연 시간 최소화를 목표로 하는 collector로 generation의 구분이 없었으나 최근부터 generation의 구분을 하기 시작했다 