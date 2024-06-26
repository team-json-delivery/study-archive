## Chapter 29. 결합도

그에 따르면 비싼 프로그램에는 모두 한 가지 공통점이 있다는 것을 발견했습니다.  
한 요소를 변경하려면 다른 요소도 변경해야 한다는 것이죠.  
반면 저렴한 프로그램은 좁은 범위의 코드 변경만 필요한 경향이 있었습니다.  

그들은 이 변경 감염 특성을 '결합도'라고 불렀습니다.  
한 요소를 변경하면 다른 요소도 함께 변경해야 하는 경우, 두 요소는 특정 변경과 관련하여 서로 결합되어 있는 것입니다.  

결합도 분석은 단순히 프로그램의 소스코드를 보는 것만으로는 부족합니다.  
두 요소가 결합되어 있는지 여부를 판단하려면, 먼저 어떤 변경이 발생했거나 발생할 가능성이 있는지 알아야 합니다.  
`테스트 해보려면 하나의 커밋에서 어떤 파일 쌍이 함께 나타나는 경향이 있는지 살펴보세요. 그런 파일들은 결합되어 있습니다.`

### 일대다 (1-N)

어떤 변경이 일어나면, 한 요소는 여러 요소와 결합이 일어납니다.  

### 연쇄작용

일단 변경이 일어나면, 한 요소에서 다른 요소로 변경이 파급되고, 그 변경은 그 자체로 또 다른 변경을 촉발하고, 스스로도 변경을 촉발할 수 있습니다.  

호출하는 곳이 하나든 1천 개든 비용은 같습니다.  

연쇄적인 변경이 더 큰 문제입니다.  

실제로 시스템이 '복잡하다'는 것은 변화가 예상치 못한 결과를 초래한다는 의미입니다.  
서비스를 담당하는 두 팀이 서로를 알지 못 했음에도 두 서비스는 백업 정책 변경과 관련하여 서로 연결되어 있었습니다.  

`가끔 지저분한 변화를 바라보고 있을 때, 결합도 때문에 마음이 흔들릴 때가 있습니다.`  

> 테스트 코드 작성할 때도 결합도를 생각해야 함  
> 복잡한 엔티티가 필요한 테스트 코드에서 엔티티를 생성하기 위해 Fixtures를 만들고, 서비스 클래스를 넣어놓은 프로젝트가 있음  
> 이로 인해 모든 테스트 코드가 @SpringBootTest를 선언해줘야 하고, 테스트 코드 실행시간도 오래 걸림  
> 가장 큰 문제는 Fixtures의 복잡도 때문에 테스트 코드를 수정할 수가 없음  
> 처음에는 복잡한 엔티티를 편하게 생성하기 위함 이었겠지만, Fixtures에 계속 추가되면서 결합도가 높아지는 문제 발생  
