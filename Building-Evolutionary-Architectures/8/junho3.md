# CHAPTER 8 진화적 아키텍처의 함정과 안티패턴

소프트웨어 프로젝트에서 발견되는 그릇된 엔지니어링 관행은 함정과 안티패턴이라는 두 가지 형태로 구분할 수 있다.  

소프트웨어 안티패턴의 두 가지 특성
- 처음에는 괜찮아 보이지만 나중에 가면 실수였으이 밝혀진다.  
- 대부분 더 나은 대안이 존재한다.  

함정은 표면적으로 괜찮은 아이디어처럼 보였다가 곧바로 나쁜 본색을 드러낸다는 점이 다르다.  


## 8.1 기술 아키텍처

### 8.1.1 안티패턴 마지막 10%의 덫, 로우코드/노코드

4GL 언어로 빠르게 개발할 수 있는 달성률은 최대 80%  
실행 스크립트, 메서드 체인 등을 동원해도 최대 90%  
결과적으로 문제를 완전히 해결하지 못 하고, 마지막 10%의 덫에 빠진다.  
4GL을 사용하면 간단한 결과물을 빠르게 만들어 낼 수 있지만, 현실 세계에 발생하는 요구조건을 충족시키기에는 역부족이다.  
개발자들은 결국 번용 언어로 되돌아가는 수밖에 없었다.  

> 4GL 언어  
> 비주얼베이직, 비주얼 C++, 파워빌더, 델파이 등  
> https://m.blog.naver.com/chuck2013/220717999167  

로우코드/노코드는 풀스택 개발은 물론 오케스트레이터처럼 특화된 도구까지 아우르는 개발 환경이다.  

> 로우코드/노코드  
> 프로그래밍 코드가 아니라 템플릿으로 개발하는 방식으로 워드프레스로 이해함  
> https://www.samsungsds.com/kr/insights/nc-lc-tech.html  

아키텍트는 일부 특수한 상황에서 로우코드 환경이나 도구를 고려할 수 있지만, 그에 따른 한계를 사전에 인지하고 생태계에 어떤 영향을 미칠지 판단해야 한다.  

> 첫 회사 홈페이지는 워드프레스로 되어 있었음  
> php로 되어있지만 문제가 발생하거나 기능을 추가하고 싶어도 개발자는 대응 불가  


### 8.1.2 사례연구 PenultimateWidgets의 컴포넌트 재사용

PenultimateWidgets의 아키텍트는 재사용성이라는 목표를 달성했지만 한편으로 새로운 병목 현상을 초래했다.  
재사용성은 개발자가 새로운 것을 빠르게 구축할 수 있다는 이점을 낳는다.  
그러나 컴포넌트팀이 동적 균형의 혁신 속도를 따라잡지 못한다면, 기술 아키텍처 컴포넌트의 재사용성은 결국 안티패턴으로 전락하는 최후를 맞는다.  

> 슬랙 / 카프카 메세지 발행을 각 애플리케이션이 담당할 것인가? 아니면 슬랙 / 카프카 메세지 발행을 담당하는 애플리케이션을 운영할 것인가?  
> 슬랙 / 카프카 메시지 발행 자체는 단순한 기능이라 재사용이 가능하고, 카프카 메세지 발행을 담당하는 애플리케이션을 운영하면 네트워크 보안 관리 측면에서 이점이 있음  
> 하지만, SPOF 문제와 기능을 커스터마이징하기 어려움  

재사용 가능한 자산을 구축할 필요가 없다는 뜻이 아니다.  
이러한 자산이 가치를 창출하고 있는지 지속적으로 평가해야 한다는 뜻이다.  

1. 커플링 지점이 진화를 방해하거나 중요한 아키텍처 특성을 저해할 경우 포크 또는 복제를 통해 커플링을 끊어야 한다.  
2. 아키텍트는 아키텍처의 각종 특성을 평가하며 이들이 지속적으로 가치를 더하고 안티패턴으로 남겨지지 않도록 관리해야 한다.  

구축 당시 아키텍트가 내렸던 올바른 결정은 시간이 흐른 뒤 틀린 결정으로 뒤바뀌는 경우가 많다.  
원래의 결정은 틀리지 않았지만 생태계가 예상치 못한 방향으로 움직인 것이다.  

> 장기적인 개발 로드맵은 있어야하고, 구성원들은 반드시 로드맵을 이해할 필요가 있다고 생각 함  
> 그래야 그 당시의 배경과 장기적인 모습 사이의 트레이드 오프를 결정할 수 있지 않을까?  
> 컬리에서는 탈고도라는 명확한 목표가 존재했고, 구성원들은 잠재적으로 목표를 달성하도록 업무를 이어나갔음  
> 카카오페이손해보험은 장기적인 로드맵이 없는 듯. 춘추전국시대


### 8.1.3 안티패턴 벤더 킹

벤더 킹 안티패턴은 특정 벤더를 중심으로 구축된 아키텍처를 의미하며, 조직과 도구가 병적인 수준의 결합을 이룬다는 특징이 있다.  
벤더 소프트웨어를 구입한 회사는 벤더가 제공하는 플러그인을 이용해 자사 비즈니스의 핵심 기능을 구현하려 한다.  
그러나 대부분의 ERP 도구는 원하는 기능을 정확히 구현할 정도로 자유롭게 가공할 수 없다.  
아키텍트는 벤더를 아키텍처의 왕으로 추대하고, 이후로는 벤더의 결정을 따르게 된다.  

아키텍처의 중심에 외부 도구나 프레임워크가 배치되면 기술적인 측면과 비즈니스 프로세스 측면에서 개발자의 능력은 심각하게 제한된다.  
대규모로 캡슐화된 도구는 비즈니스 관점에서 결국 마지막 10%의 덫을 피할 수 없다.  

아키텍처와 벤더 킹이 커플링을 생성하지 않도록 주의하라  

밴더 킹 안티패턴의 희생양이 되지 않으려면 벤더 제품을 독립적인 통합 지점으로 취급해야 한다.  

> 토스ㅣSLASH 23 - 새로운 은행을 위한 Modern 대외 연계 시스템 구축기  
> https://youtu.be/eS9tukmYBLI?si=D2H_GnnknLyQIeWw  


### 8.1.4 함정 유출된 추상화

추상화가 복잡할수록 이를 구현한 상세 정보가 일정 부분 유출될 가능성이 크다.  

개발자는 취상화가 항상 정확하다고 믿고 싶어 하지만, 추상화는 종종 깜짝 놀랄만한 방식으로 망가지며 개발자를 당황케 한다.  

> 점진적 추상화 | 인프콘2023  
> https://youtu.be/dzDCToa0XNg?si=gROF6XSMWSgY3pVn  

근래 들어 기술 스택의 복잡도가 증가하며 추상화 방해 문제도 덩달아 심화되었다.  

원시 추상화 오염은 낮은 수준의 추상화가 깨지며 예측할 수 없는 혼란을 일으키는 현상을 뜻한다.  
이는 기술 스택의 복잡도가 증가할 때 생기는 대표적인 부작용이다.  

일상적으로 접하는 계층보다 더 낮은 수준의 추상화 계층을 한 개 이상 완벽히 이해하라.  

기술 스택의 복잡도 증가는 동적 평형이 유발하는 부작용이다.  
시간이 흐르면 생태계가 변할뿐만 아니라 컴포넌트는 점점 복잡해지며 서로 얽히게 된다.  
진화적 변화를 보호하는 메커니즘, 즉 피트니스 함수는 아키텍처의 취약한 결합 지점을 보호할 수 있다.  


### 8.1.5 함정 이력서 주도 개발

효과적인 아키텍처를 설계할 때는 신기술을 들추기보다 도메인을 면밀히 살피고 필요한 기능을 파악하는 것이 먼저다.  
아키텍트가 이력서 주도 개발이라는 함정에 빠졌다면 이야기가 달라진다.  
이력서에 경력을 추가하기 위해 가능한 한 모든 프레임워크와 라이브러리를 아키텍처에 동원하려 할 것이다.  

> 기술은 수단이지, 목적이 되면 안 됨  
> 양심고백하자면, 컬리 마지막에 PHP에서 코틀린으로 넘어간건 함정 이력서 주도 개발이었음  


## 8.2 증분 변경

### 8.2.1 안티패턴 부적절한 거버넌스

소프트웨어 아키텍처는 외부와 단절된 존재가 아니며, 종종 자신이 설계한 주변 환경을 비추어 보여주곤 한다.  
공유 리소스를 극대화하고 비용을 절감하기 위한 환경은 그에 맞는 거버넌스 모델을 발전시켰다.  
그러나 여러 리소스를 하나의 머신에 압축하는 방식은 개발 관점에서 볼 때 바람직하지 않다.  
공유 리소스를 아무리 효과적으로 격리한다 해도 결국 언젠가는 리소스 경합이 발생하기 때문이다.  

> 요즘은 스마트폰도 하드웨어가 워낙 좋아서 예전만큼 리소스 최적화 개발은 줄어들지 않았을까?  
> 물론 최적화 실패시, 발열과 배터리 사용 문제가 있긴할 듯  

이제 개발자는 마이크로서비스처럼 컴포넌트 격리 수준이 높은 아키텍처를 구축할 수 있다.  

포브스의 유명한 선언을 풀어서 설명하면, 항공사의 아이패드 앱이 부실하면 결과적으로 항공사의 수익이 저하된다는 뜻이다.  

마이크로서비스 아키텍처의 공통된 특징 중 하나는 폴리글랏 환경을 용인한다는 것이다.  
각 서비스팀은 기업 표준을 준수하거나 균질성을 유지할 필요가 없이, 서비스를 구현하는 데 적합한 기술 스택을 자유롭게 선택할 수 있다.  

현대의 개발 환경은 단일 기술 스택의 균질성을 유지하는 거버넌스와 어울리지 않는다.  
이러한 거버넌스는 시스템으 복잡도를 과도하게 높이며, 솔루션을 구현하기 위한 노력을 곱절로 배가시킨다.  
모놀리식 아키텍처는 거버넌스 결정이 모두에게 영향을 미친다.  

마이크로서비스는 기술적인 측면, 데이터 아키텍처 측면에서 서로 결합할 필요가 없다.  
따라서 각 팀은 자신의 서비스에 최적화된 복잡도와 정교함을 올바르게 결정할 수 있다.  
최종적인 목표는 서비스 스택의 복잡성을 기술적 요구 사항에 맞춰 단순화하는 것이다.  


#### 강제 디커플링

모든 개발자가 동일한 코드베이스나 플랫폼을 공유한다면 기존 코드를 재사용하고 싶은 유혹에 흔들리기 쉽다.  
채드 파울러는 개발자가 이식성을 확보하는 것보다 우발적인 커플링을 방지하는 것이 훨씬 중요하며, 모든 팀은 반드시 서로 다른 기술 스택을 채택해야 한다는 것이다.  

> 채용 등 현실적인 문제가 클 듯  


### 8.2.2 사례연구 PenultimateWidgets의 'Just Enough' 거버넌스

아키텍트는 자바와 오라클을 중심으로 모든 개발 환경을 표준화하려 노력했다.  
그러나 세분화된 서비스가 늘어남에 따라 이러한 스택이 소규모 서비스에 엄청난 복잡성을 부여한다는 사실을 깨달았다.  

#### 소형

#### 중형

#### 대형


### 8.2.3 함정 릴리스 속도 저하

릴리스 절차가 고정되어 있고 다양한 특수 작업을 동반한다면 진화적 아키텍처를 활용할 여지는 제한된다.  

순환 주기는 개발자가 신기능을 만들기 시작할 때부터 해당 기능이 프로덕션 환경에서 실행될 때까지의 시간이다.  
순환 주기를 측정하는 목표는 엔지니어링의 효율성을 파악하는 것이며, 순화 주기를 단축하는 것은 지속적 전달의 핵심 과제 중 하나다.  

순환 주기가 빠르다는 것은 아키텍처가 더 빠르게 진화할 수 있음을 의미한다.  
소프트웨어 릴리스 속도가 빨라질수록 시스템의 발전 속도도 빨라진다.  

순환 주기에 피트니스 함수를 적용하고 초과할 경우 경보가 울리도록 설정했다.  
피트니스 함수가 임겟값에 도달하면 개발자는 배포 파이프라인을 재구성할 것인지 순환 주기를 4시간 이상으로 늘릴 것인지 결정해야 한다.  
대부분의 프로젝트에서 개발자는 점진적으로 증가하는 순환 주기를 인식하지 못하기에, 서로 상충하는 목표의 우선순위를 저울질할 기회가 없다.  
따라서 자신도 모르는 사이에 암묵적으로 결정을 내리는 결과를 낳는다.  


## 8.3 비즈니스 관심사

비즈니스 담당자는 개발자를 괴롭히는 악당이 아니다.  
다만 그들은 아키텍처 관점에서 부적절한 결정을 내릴 수 있는 우선권을 가지고 있기에, 의도치 않게 미래의 가능성을 제한하곤 한다.  

> 개인적으로 기획자도 시스템 구조를 이해하고, 피처 개발 시 개발자와 함께 얘기해야한다고 생각 함  
> 그래야 개발자와 기획자 모두 시스템과 서비스라는 두 목표를 달성할 수 있다고 생각 함  
> 
> 컬리에서 가장 놀랐던 건 기획자가 테이블 설계를 1차로 제공했던 점, 그리고 시스템 아키텍처 이해도가 높았음  
> 카카오페이손해보험은 기획자가 정책과 화면을 정의하고, 개발자는 시스템 설계를 하는 분위기  
> 회사(팀)마다 업무 방식이 있겠지만, 기획자와 개발자가 각자의 역할만 하는건 좋지 않은 것 같음  


### 8.3.1 함정 제품 맞춤화

실행 경로가 늘어날수록 테스트의 부담은 상당히 가중된다.  


### 8.3.2 안티패턴 기록 시스템에 기반한 보고 시스템

보고서는 모놀리식 아키텍처의 우발적 커플링을 설명하기 좋은 예시다.  
아키텍트와 DBA는 기록 시스템과 보고 시스템에 동일한 데이터베이스 스키마를 사용하기를 원한다.  
문제는, 하나의 아키텍처가 둘을 모두 지원하도록 최적화시킬 수는 없다는 것이다.  
서로 상충하는 비즈니스 목표가 아키텍트의 작업을 방해하고 진화적 변화의 난이도를 높이는 전형적인 사례라 할 수 있다.  

> 모놀리식 시스템에 익숙한 개발자들은 어떻게서든 기존 테이블을 활용하려고 함  
> 그러다가 DB 장애가 발생 함  
> 테이블을 새롭게 정의하는 방법도 있으니, 기간계 테이블에 지나치게 의존하지 말자고 얘기하는 중  


### 8.3.3 함정 과도하게 긴 계획 시간

가정을 세우는 데 너무 많은 노력을 기울인 나머지 개발자는 자신의 가정에 애착을 느끼게 된다.  
이처럼 감정적 투자의 영향을 받은 결정 과정은 매몰 비용으로 설명할 수 있다.  
특정 대상에 시간과 노력을 많이 투자할수록 대상을 포기하기는 점점 더 어려워진다는 뜻이다.  

> 초반에 옳바른 가정을 세우려면 어떻게 할 수 있을까?  

대규모 프로그램은 초기부터 작은 단위로 나누어 아키텍처 결정과 개발 인프라의 타당성을 테스트하는 것이 좋다.  
아키텍트는 소프트웨어를 실제로 구축하기 전에 대규모 투자가 결정되지 않도록 주의해야 한다.  


### 요약

진화적 아키텍처 또한 기술, 비즈니스, 운영, 데이터, 통합 측면에 트레이드오프를 동반한다.  
패턴과 안티패턴은 각각이 주는 조언을 따르는 것도 중요하지만, 배경에 감추어진 맥락을 이해하는 것이 더욱 중요하다.  
아키텍트가 내리는 모든 결정은 트레이드오프를 새롭게 평가해야 한다.  
패턴과 안티패턴은 트레이드오프 상황에 맞는 조언자 역할과 안티패턴을 식별하는 안전장치 역할을 한다.  
