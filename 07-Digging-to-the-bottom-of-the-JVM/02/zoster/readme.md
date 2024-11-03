# The Java memory area and overflow
## 2.1 들어가며
* Java는 골치아픈 메모리 관리를 가상 머신에게 위임하는 대신 문제가 터진 경우 가상 머신의 메모리 관리 방식을 이해하지 못했을 때 
해결하기 어렵다는 문제가 있다

## 2.2 런타임 데이터 영역
* 자바 가상 머신이 관리하는 메모리는 여러 런타임 데이터 영역으로 구성된다

### 2.2.1 Program counter
* Program counter register는 현재 실행중인 thread의 bytecode line numbr 표시기라고 생각하면 된다
* 다음에 실행할 bytecode instruction을 선택하는 식

### 2.2.2 Java virtual machine stack
* Java virtual machine stack 역시 thread private이다
* Java의 memory 영역을 heap / stack으로 구분하는 사람이 많은데, 사실은 이보다 더 복잡하다
* Stack이라고 하면 보통 지역 변수 테이블을 가리킬 때가 많은데
  * 지역 변수 테이블에는 java virtual machine 이 compile 타임에 알 수 있는 다양한 기본 data type, object reference, return address type을 저장한다
  * 지역 변수 테이블은 compile time에 할당된다
  * Java method는 stack frame에서 지역 변수용으로 할당받아야 할 공간의 크기가 결정되어 있다

### 2.2.3 Native method stack
* virtual machine stack과 같다

### 2.2.4 Java heap
* Java heap은 모든 thread가 공유하며 가상 머신이 구동될 때 만들어진다.
* 객체 인스턴스를 저장한다(Stack에는 refernec만)

### 2.2.5 Method 영역
* Heap과 동일하게 모든 스레드가 공유한다
* virtual machine이 읽어들인 type, constant, static, JIT compiler의 code cache 등이 저장된다
* Method를 permanent로 부르며 혼동 했지만, hostpot 팀에서 method를 permanent 영역에 구현했을 뿐이다
* Java 8부터는 Permanent Generation(영구 영역)이 Metaspace로 대체되었다. 따라서 -XX:MaxPermSize 옵션은 Java 8 이상에서 사용할 수 없으며, Metaspace 크기는 -XX:MaxMetaspaceSize로 설정할 수 있음

### 2.2.6 Runtime constant pool
* Method 영역의 일부
* Class version, field, method, interface 등 class 파일에 포함된 설명에 더해 compile time에 생성된 literal과 symbol 참조가 저장된다
* Virtual machine이 class load시에 이 정보를 method 영역의 runtime constant pool에 저장한다

### 2.2.7 Direct memory
* Virutal machine runtime에 속하지 않는다
* NIO를 통해 heap이 아닌 memory를 직접 할당할 수 있는데 DirectByteBuffer 객체를 통해 작업을 수행할 수 있다
* 이로 이내 Java heap <-> Native heap 사이에 데이터를 복사하지 않아도 돼 일부 시나리오에서 큰 성능 향상이 있다
* 다만 java의 Heap 크기와 무관할뿐 간과하면 OOM 이 발생할 수 있다. `OutOfMemoryError: Direct buffer memory`

## 2.3 Hotspot virtual machine에서의 객체 들여다보기
### 2.3.1 객체 생성
* new 키워드를 사용하여 객체를 만들 때 어떤 일이 일어나는가
  1. 상수 풀 안의 클래스를 가리키는 symbol reference 인지 확인한다
  2. reference 하고자 하는 클래스가 loading, resolve, initialize 되었는지 확인한다
  3. 준비되지 않았다면 loading 한다
  4. loading이 완료된 클래스라면 새 객체를 담을 메모리를 allocation한다
  > singletone이라면 reference만 반환하면 될듯
  5. allocation이 완료되면 0으로 초기화 한다(이로 인해 인스턴스 필드가 0,null이 된다) : 초기화하지 않으면 쓰레기값이 memory에 남아있을 수 있기 때문
  6. `각 객체에 필요한 설정` 을 한다. 객체 헤더에 저장된다
* Memory allocation 과정에서 race condition이 발생하면?
  * memory allocation을 synchonize해서 처리하는 방법
  * thread 별로 다른 memory 공간을 할당하는 방법(TLAB : Thread Local Allocation Buffer), -XX:+/-UseTLAB로 설정 가능(기본 true)

### 2.3.2 객체의 memory layout
* Hotspot virtual machine 은 객체를 세 부분으로 나눠 heap에 저장한다
* object header, instance data, alignment padding

#### Object header
* 두 가지 유형의 데이터를 담고 있다
1. 객체의 runtime 데이터 : Hash code, GC generation, Lock, thread lock, biased thread, biased 시각의 타임스탬프
   2. Mark word라고 하며 virtual machine의 bit 크기를 따른다
3. Klass word : 클래스 관련 메타데이터의 포인터가 저장된다
4. 객체가 배열인 경우 배열의 길이도 헤더에 저장한다, 메모리 계산을 위해 type도 함께 header에 저장한다

#### Instance data
* 객체가 실제로 담고 있는 정보
* field의 정보, parent class 유무, parent class에 정의한 field 들
#### Alignment padding
* 객체의 크기가 8 byte가 아닌 경우 맞춰주기 위한 padding

### 2.3.3 객체에 접근하기
* 객체 접근은 주로 Handle이나 direct pointer를 사용해 구현한다
* handle에는 객체의 인스턴스 타입, 데이터등 주소 정보가 담기고 인스턴스에는 handle의 주소가 저장된다
  * 장점은 handle이 garbage collection의 영향을 받지 않아 GC 후에 수정범위가 작다는 장점이 있다
* Direct pointer 방식은 heap에 있는 개체에서 인스턴스 데이터뿐 아니라 타입에 접근하는 path도 제공해야 한다
  * handle을 경유하는 오버헤드가 없으므로 속도가 빠르다

## 2.4 실전 : OOM exception
### 2.4.1 Java heap overflow
* 객체를 저장하므로 계속해서 객체가 생성되고 제거되지 않는다면 overflow가 된다
* heap에서 발생하면 `java.lang.OutOfMemoryError` 옆에 `Java heap space`라고 출력된다
* 해결하는 일반적인 방법은 Memory image analyzer로 heap dump snapshot을 분석하는 것이다. 
  * [eclipse MAT](https://eclipse.dev/mat/)
* 꼭 필요한 객체(메모리 자체가 프로그램의 요구사항보다 작은 경우)가 아니라면 Memory leak이다
  * > MAT의 사용법이 그리 어렵지 않으므로 이것저것 눌러보다 보면 뭐가 원인인지 알게 됨, 할당된 memory 크기, 객체의 수, code line 등이 모두 표현됨

### 2.4.2 Virtual machine stack과 Native method stack overflow
* Hotspot은 virtual machine stack과 native method stack을 구분하지 않는다
* 일반적으로 Stack에서 StackOverflow보다 OOM이 먼저 발생하긴 어렵다

### 2.4.3 Method 영역과 Runtime constant pool overflow
* 두 영역 모두 같은 영역에 속하므로 함께 테스트할 수 있다
* JDK 7 이전에는 Constant pool이 Permanent에 할당되었으므로 string.inter()을 반복해서 호출하면 OOM이 발생한다
* 그러나 JDK 7 부터는 같은 코드를 수행해도 heap으로 옮겨졌기 때문에 문제가 없다(GC가 처리하기 때문)

### 2.4.4 Native direct memory overflow
* 억지 코드