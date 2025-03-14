# 3장. 가비지 컬렉터와 메모리 할당 전략
## 3.1 들어가며
- 자바는 모든 객체 인스턴스가 힙에 저장된다.
- GC가 힙을 청소하려면 객체가 살아 있고, 죽었는지 판단해야 한다.
- 죽었다는 의미는 프로그램에서 더이상 사용하지 않음을 의미한다.

## 3.2 대상이 죽었는가?
- 3.2.1 참조 카운팅 알고리즘
  - 객체가 살아 있는지 판단하는 알고리즘 (JVM은 사용하지 않음)
  - 객체를 가리키는 참조 카운터를 추가한다. 참조하는 곳이 하나 늘어날 때마다 카운터 값을 1씩 증가시킨다. 
  - 참조하는 곳이 하나 사라질 때마다 카운터 값을 1씩 감소시킨다.
  - 카운터 값이 0이 된 객체는 더는 사용될 수 없다.
- 3.2.2 도달 가능성 분석 알고리즘
  - 자바, C# 등 오늘날의 주류 프로그래밍 언어들은 모두 객체 생사 판단에 도달 가능성 분석 알고리즘을 이용한다.
  - GC 루트라고 하는 루트 객체들을 시작 노드 집합으로 쓰고, 참조하는 다른 객체들을 탐색한다. (탐색 과정에서 만들어지는 경로를 참조 체인)
  - 어떤 객체와 GC 루트 사이를 이어 주는 참조 체인이 없다면, 즉 GC 루트로부터 도달 불가능한 객체라고 판단하고 제거
  - 자바에서 GC 루트로 이용할 수 있는 객체 
    - 가상 머신 스택(스택 프레임의 지역 변수 테이블)에서 참조하는 객체: 현재 실행중인 메서드에서 쓰는 매개 변수, 지역 변수, 임시 변수 등 
    - 메서드 영역에서 클래스가 정적 필드로 참조하는 객체 : 자바 클래스의 참조 타입 정적 변수 
    - 메서드 영역에서 상수로 참조되는 객체 : 문자열 테이블 안의 참조 
    - 네이티브 메서드 스택에서 참조하는 객체 
    - 자바 가상 머신 내부에서 쓰이는 참조 : 기본 데이터 타입에 해당하는 Class 객체, 일부 상주 예외 객체, 시스템 클래스 로더 
    - 동기화 락으로 잠겨 있는 모든 객체 
    - 자바 가상 머신 내부 상황을 반영하는 JMXBean: JVMTI에 등록된 콜백, 로컬 코드 캐시
- 3.2.3 다시 참조 이야기로
  - 강한 참조 : 가장 전통적인 정의의 참조, Object object = new Object()처럼 프로그램 코드에서 참조를 할당하는 걸 의미, 강한 참조 관계가 남아 있는 객체는 GC가 절대 회수 안함
  - 부드러운 참조 : 유용하지만 필수는 아닌 객체를 표현, 부드러운 참조만 남은 객체라면 메모리 오버플로가 나기 직전에 두 번째 회수를 위한 회수 목록에 추가
  - 약한 참조 : 부드러운 참조와 비슷하지만 연결 강도가 더 약함. 약한 참조뿐인 객체는 다음번 GC까지만 생존, GC가 동작하면 메모리가 넉넉하더라도 약하게 참조된 객체는 모두 회수
  - 유령 참조 : 참조 중에 가장 약하고, 객체 수명에 아무런 영향을 주지 않으며, 유일한 목적은 객체가 회수될 때 알림을 받기 위해서임
  - 파이널 참조 : 약한 참조와 유령 참조 사이, finalize() 메서드를 구현한 객체는 파이널 참조 대상이 되어 별도 대기열에 등록
- 3.2.4 살았나 죽었나?
  - 도달 가능성 분석 알고리즘이 도달 불가능으로 판단해도 바로 죽는건 아니고 유예 단계가 있음
  - 도달 가능성 분석으로 참조 체인을 찾지 못한 객체에는 첫번째 표시가 이루어지며 필터링이 진행됨. 필터링 조건은 finalize() 메서드를 실행해야 하는 객체인가, 필요 없는 객체이거나 이미 호출한 경우 모두 실행할 필요 없음으로 처리
  - finalize()를 실행해야 하는 객체로 판명되면 F-Queue라는 대기열에 추가
  - finalize로 한번은 살 수 있다. 
  - https://jaeyeong951.medium.com/finalize-%EC%9D%80%ED%87%B4%EC%8B%9D-4a52fb855910
  - https://www.baeldung.com/java-18-deprecate-finalization
- 3.2.5 메서드 영역 회수하기
  - 메서드 영역의 가비지 컬렉션은 더 이상 사용되지 않는 상수와 클래스를 회수한다.
  - 상수를 회수하는 방법은 힙에서 객체를 회수하는 방법과 비슷하다. (참조가 없다면 삭제한다.)
  - 클래스
    - 이 클래스의 인스턴스가 모두 회수되었다. (힙에 해당 클래스와 하위 클래스의 인스턴스가 하나도 존재하지 않는다.)
    - 클래스 로더가 회수되었다.
    - 이 클래스에 해당하는 java.lang.Class객체를 아무 곳에서 참조하지 않고, 리플렉션 기능으로 이 클래스의 메서드를 이용하는 곳도 없다.
      위처럼 회수는 할 수 있지만, 힙에 비해 가성비가 안좋다.

## 3.3 가비지 컬렉션 알고리즘
- 객체의 생사를 판별하는 방식을 기준으로 가비지 컬렉션 알고리즘은 참조 카운팅 GC와 추적 GC로 나뉜다.
- 세대 단위 컬렉션 이론의 가설
  - 약한 세대 가설 : 대다수 객체는 일찍 죽는다.
  - 강한 세대 가설 : 가비지 컬렉션 과정에서 살아남은 횟수가 늘어날수록 더 오래 살 가능성이 커진다.
  - 세대 간 참조 가설 : 세대 간 참조의 개수는 같은 세대 안에서의 참조보다 훨씬 적다.
- 세대 단위 컬렉션 이론을 가상 머신에 적용한 설계자들은 자바 힙을 최소 두 개 영역(신세대, 구세대)로 나눈다.
- 신세대에서는 가비지 컬렉션 때마다 다수의 객체가 죽고 살아남은 소수만 구세대로 승격된다.
- 상호 참조 관계의 두 객체는 삶과 죽음을 함께하는 경향이 있다.
- 신세대 객체가 구세대 객체를 참조하면, 구세대 객체는 잘 죽지 않아서 신세대 객체는 곧 구세대로 승격이 될 거고, 그러면서 같은 세대가 된다.
- 세대 간 참조의 수는 아주 적기 때문에 구세대 전체를 훑는 건 낭비 -> 신세대에 기억 집합이라는 전역 데이터 구조를 하나 두고, 세대 간 참조가 있는지 기록해 관리하는게 효율적
- GC 종류
  - 마이너 GC : 신세대만 대상으로 하는 가비지 컬렉션
  - 메이저 GC : 구세대를 대상으로 하는 가비지 컬렉션. (CMS 컬렉터만 구세대를 따로 회수)
  - 혼합 GC : 신세대 전체와 구세대 일부를 대상으로 하는 가비지 컬렉션 (G1 컬렉터)
  - 전체 GC : 자바 힙 전체와 메서드 영역까지 모두를 대상으로 하는 가비지 컬렉션
- 3.3.2 마크-스윕 알고리즘
  - 작업을 표시(Mark)와 쓸기(Sweep)라는 두 단계로 나눠 진행
  - 회수할 객체들에 모두 표시한 다음, 표시된 객체들을 쓸어 담는 식, 반대로 살릴 객체에 표시하고 표시되지 않은 객체를 회수
- 3.3.3 마크-카피 알고리즘
  - 가용 메모리를 똑같은 크기의 두 블록으로 나눠 한 번에 한 블록만 사용, 한쪽 블록이 꽉 차면 살아남은 객체들만 다른 블록에 복사하고, 기존 블록을 한번에 청소
  - 복사 과정에서 객체들이 메모리의 한쪽 끝에서부터 차곡차곡 쌓이기 때문에 메모리 파편화 문제로부터 해방
  - 가용 메모리를 절반으로 줄여 낭비가 심하다는 문제가 있는데, 신세대 객체 중 98%가 첫 번째 가비지 컬렉션에서 살아남지 못한다는 통계를 바탕으로 아펠 스타일 컬렉션 방식이 등장
  - 아펠 스타일 컬렉션 방식 
    - 신세대를 하나의 큰 에덴 공간과 두 개의 작은 생존자 공간으로 나누고 메모리를 할당할 때는 생존자 공간 중 하나와 에덴만 사용.
    - 가비지 컬렉션이 시작되면 에덴과 생존자 공간에서 살아남은 객체들을 나머지 생존자 공간으로 하나씩 복사한 후 이전 생존자 공간을 곧바로 비운다.
    - 핫스팟 가상 머신에서 에덴과 생존자 공간의 비율은 기본적으로 8:1, 신세대에 할당된 전체 메모리 중 90%를 활용
    - 마이너 GC에서 살아남은 객체를 생존자 공간이 다 수용하지 못할 경우 다른 메모리 영역(구세대)에 메모리 할당
- 3.3.4 마크-컴팩트 알고리즘
  - 마크-카피 알고리즘은 객체 생존율이 높을수록 복사할 게 많아 효율이 나빠지기에 구세대에는 적합하지 않다. 
  - 표시한 후 컴팩트 단계에서 생존한 모든 객체를 메모리 영역의 한쪽 끝으로 모은 다음, 나머지 공간을 한꺼번에 비우는 알고리즘
## 3.4 핫스팟 알고리즘 상세 구현
- 3.4.1 루트 노드 열거
  - 도달 가능성 분석 알고리즘에서 GC 루트 집합으로부터 참조 체인을 찾는 작업
  - GC 루트로 고정할 수 있는 노드는 주로 전역 참조(상수와 클래스 정적 속성 등)와 실행 콘텍스트(스택 프레임의 지역 변수테이블)에 존재
  - 도달 가능성 분석 알고리즘에서 가장 오래 걸리는 참조 체인 찾기 과정은 사용자 스레드와 동시에 실행이 가능하나, 루트 노드 열거만큼은 반드시 일관성이 보장되는 스냅숏 상태에서 수행되어야한다.(사용자 스레드 정지)
  - 루트 노드 열거때문에 가비지 컬렉션 시 모든 사용자 스레드가 일시 정지해야 한다.
- 3.4.2 안전 지점
  - 핫스팟은 OopMap을 활용하여 GC 루트들을 빠르고 정확하게 열거 가능 
  - 핫스판은 안전 지점이라고 하는 특정한 위치에서 OopMap을 갱신하며, 가비지 컬렉터는 사용자 프로그램이 안전 지점에 도달할 때까지는 절대 멈춰 세우지 않는다.
  - 안전 지점의 위치를 선택하는 기준은 기본적으로 프로그램이 장시간 실행될 가능성이 있는가이다. 
  - 장시간 호출되는 대표적인 예시 : 메서드 호출, 순환문, 예외 처리 등 명령어 흐름을
- 3.4.3 안전 지역
  - 안전 지점 메커니즘은 실행 중인 프로그램이 그리 길지 않은 시간에 안전 지점에 도달하여 가비지 컬렉션 프로세스가 제대로 임무를 다할 수 있게끔 보장한다.
  - 안전 지역은 일정 코드 영역에서는 참조 관계가 변하지 않음을 보장한다. 안전 지역 안이라면 어디서든 가비지 컬렉션을 시작해도 무방
- 3.4.4 기억 집합과 카드 테이블
  - 가비지 컬렉터는 신세대에 기억 집합이라는 데이터 구조를 두어 객체들의 세대 간 참조 문제를 해결한다. 
  - 구세대와 GC 루트 전부를 스캔해야 하는 사태를 기억 집합을 이용하여 방지
  - 기억 집합은 비회수 영역(회수 대상이 아닌 영역)에서 회수 영역을 가리키는 포인터들을 기록하는 추상 데이터 구조
  - 카드 페이지 하나의 메모리에는 보통 하나 이상의 객체가 들어가 있는데, 이 객체들 중 하나에라도 세대 간 포인터를 갖는 필드가 있다면, 1로 표시한다.(더렵혀진다)
  - 객체를 회수할 때는 카드 테이블에서 더럽혀진 원소만 확인하면 어떤 카드 페이지의 메모리 블록이 세대 간 포인터를 포함하는지 쉽게 파악 가능
- 3.4.5 쓰기 장벽
  - 다른 세대의 객체가 현 블록 안의 객체를 참조하면 카드 테이블의 해당 원소가 더럽혀지는데, 더럽혀지는 시점은 참조 타입 필드에 값이 대입되는 순간
    핫스팟 가상 머신은 쓰기 장벽 기술을 이용해 카드 테이블을 관리한다.
  - 쓰기 장벽 기술은 AOP와 같다. 대입 전 쓰기 장벽을 사전 쓰기 장벽, 대입 후 쓰기 장벽을 사후 쓰기 장벽
  - 가상 머신은 쓰기 장벽을 활용해 카드 테이블을 최신화
- 3.4.6 동시 접근 가능성 분석
  - 삼색 표시 기법 
    - 흰색 : 가비지 컬렉터가 방문한 적 없는 객체, 도달 가능성 분석을 시작하면 처음에는 모든 객체가 흰색, 분석 마친 뒤에도 흰색인 객체는 도달 불가능한 객체
    - 검은색 : 가비지 컬렉터가 방문한 적 있으며, 이 객체를 가리키는 모든 참조를 스캔함. 검은 객체는 스캔되었고 확실히 생존함을 뜻한다. 다른 객체에서 검은 객체를 가리키는 참조가 있다면 다시 스캔하지 않아도 된다. 
    - 회색 : 가비지 컬렉터가 방문한 적 있으나, 이 객체를 가리키는 참조 중 스캔을 완료하지 않은 참조가 존재
  - 스냅샷 상태에서 루트 노드 열거를 진행해야 하는 이유 
    - 죽은 객체를 살았다고 잘못 표시할 수 있다. 이는 좋지 않은 일이지만 그래도 감내할 수 있다. 다음번 GC때 청소됨
    - 살아 있는 객체를 죽었다고 표시할 수 있다. 이는 아주 치명적인 오류다. 이러한 오류는 스캔 도중 객체가 사라질 때 일어난다.

## 3.5 클래식 가비지 컬렉터
- 3.5.1 시리얼 컬렉터
  - 단일 스레드로 동작하는 컬렉터 
  - 가비지 컬렉션이 시작되면 회수가 완료될 때까지 다른 모든 작업 스레드가 멈춰 있어야 한다.
  - 시리얼 컬렉터는 다른 컬렉터의 단일 스레드 알고리즘보다 간단하고 효율적 
  - 시리얼 컬렉터를 사용하고 싶다면 -XX:+UseSerialGC 매개 변수 추가
- 3.5.2 파뉴 컬렉터
  - 파뉴 컬렉터는 여러 스레드를 활용하여 시리얼 컬렉터를 병렬화한 버전
  - 시리얼 컬렉터를 제외하고는 CMS 컬렉터와 조합하여 사용할 수 있는 컬렉터 
  - G1의 등장으로 힘이 약해짐
- 3.5.3 패러렐 스캐빈지 컬렉터
  - PS 컬렉터라고 불리는데, 처리량을 제어하는게 목표. 처리량이란 프로세서가 사용자 코드를 실행하는 데 사용하는 시간과 프로세서가 소비하는 총 시간의 비율 처리량 = 사용자 코드 실행 시간 / 사용자 코드 실행 시간 + 가비지 컬렉터 실행 시간
  - 가비지 컬렉션 정지 시간의 최댓값을 지정하는 -XX:MaxGCPauseMillis와 처리량을 직접 지정하는 -XX:GCTimeRatio VM 매개변수가 존재
- 3.5.4 시리얼 올드 컬렉터
  - 구세대에서 사용하는 시리얼 컬렉터
  - 단일 스레드 컬렉터이며 구세대이니까 마크-컴팩트 알고리즘을 사용한다.
- 3.5.5 패러렐 올드 컬렉터
  - PS 컬렉터의 구세대용
  - 멀티스레드를 이용한 병렬 회수를 지원하며 마크-컴팩트 알고리즘을 기초로 구현
- 3.5.6 CMS 컬렉터
  - 표시와 쓸기 단계 모두를 사용자 스레드와 동시 수행 
  - CMS 컬렉터의 목적은 가비지 컬렉션에 따른 일시 정지 시간을 최소로 줄이는 것
  - CMS 단점 
    - 프로세서 자원에 아주 민감하다. 동시 수행 단계에서 사용자 스레드를 멈추지는 않더라도 애플리케이션을 느리게 하고 전체 처리양르 떨어뜨리는 건 피할 수 없다.
    -CMS가 부유 쓰레기를 처리하지 못해서 동시모드 실패를 유발할 가능성이 있다. 동시 모드 실패가 발생하면 전체 GC 발생 
    - 마크-스윕 알고리즘이기에 메모리 파편화가 발생한다.
- 3.5.7 G1 컬렉터
  - 가비지 우선 컬렉터, G1은 부분 회수라는 컬렉터 설계 아이디어와 리전을 회수 단위로 하는 메모리 레이아웃 분야를 개척
  - G1은 정지 시간 예측 모델을 사용해, 목표 시간을 M 밀리초로 설정하면 가비지 컬렉터가 쓰는 시간을 M 밀리초가 넘지 않도록 통제한다.
  - G1은 힙 메모리의 어느 곳이든 회수 대상에 포함되며, 이를 회수집합 CSet이라 한다.
  - 어느 세대에 속하느냐가 아니라 어느 영역에 쓰레기가 가장 많으냐와 회수했을 때 이득이 어디가 가장 크냐가 회수 영역을 고르는 기준
  - G1은 연속된 자바 힙을 동일 크기의 여러 독립 리전으로 나누고, 각 리전은 신세대의 에덴, 생존자, 구세대가 될 수 있다.
  - G1 출시 문제점 
    - 자바 힙을 다수의 독립된 리전으로 나눈다면 객체들의 리전 간 참조 문제 해결이 필요(원래는 구세대 신세대)
    - 동시 표시 단계 동안 GC 스레드와 사용자 스레드가 서로 간섭하지 않도록 보장 필요
- 3.5.8 오늘날의 가비지 컬렉터들
  - 통폐합.
  - JDK9~ 시리얼/패러럴/G1
  - JDK15~ ZGC (JDK21~ 세대구분 ZGC)
  - JDK12~ 셰넌도어