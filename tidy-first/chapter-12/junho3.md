## Chapter 12. 도우미 추출

코드 블록을 추려내고, 도우미로 추출한 후에 이름을 붙입니다.  
**이때, 도우미의 이름은 작동 방식이 아니라 목적에 따라 짓습니다.**  

리팩토링을 알고 계신 분들이라면, 이 정리법이 바로 '메서드 추출' 리팩토링임을 알아볼 것입니다.  

해당하는 줄들을 도우미로 추출하고, 도우미 안의 내용만 변경한 다음, 적절하다고 판단한 뒤에 도우미를 호출하는 문장에 반영하세요.  
이 작업을 응집도 높은 요소 만들기로 이해하실 수 있을 것입니다.  

도우미 추출의 또 다른 특수 사례는 시간적 결합을 표현하는 경우입니다.  
이럴 때는 다음(ab())처럼 바꾸세요.  

도우미는 필요한 모든 곳에서 사용할 수 있습니다.  
도우미 사용은 또 다른 코드 정리에 도움이 됩니다.  

> 코틀린은 object 키워드로 싱글턴 클래스를 쉽게 만들 수 있음  
> object 클래스로 helper를 정의하는 편  
> 함수 또한 public으로 선언할 수 있어서 테스트 코드를 작성할 수 있음  
> 
> Java에서 도우미 메소드를 private으로 정의하는지? 별도 클래스로 정의 후 주입해서 사용하는지?  
>
> if (condition1 == true || condition2 == true) { ... } (x)  
> if (isCondition1() || isCondition2()) { ... } (x)  
> if (isPayAble()) { ... } (o)  
> isCondition1(), isCondition2()과 같이 너무 작은 도우미 함수는 의미 없다고 생각함  
> 
> if (condition) { doSomething() } vs doSomething(condition)  
> 조건을 만족하는 경우에 도우미 함수를 호출하는지? 아니면 도우미 함수에 조건 자체를 넘기는지?  
>
> isCondition1(), isCondition2()과 같이 너무 작은 도우미 함수는 의미 없다고 생각함  
> 도우미 함수도 누군가가 유지 보수해야함  
> value class Money (val value: BigDecimal) {  
>   fun toInt() { ... }  
>   fun toLong() { ... }  
> }  
> Money.toInt() vs Money.value.toInt()
