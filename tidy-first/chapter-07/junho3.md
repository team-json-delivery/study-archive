## Chapter 07. 선언과 초기화를 함께 옮기기

타입이 포함된 선언과 초기화 코드가 떨어져 있기라도 하면 읽어 내기는 더 어려워집니다.  
한참 뒤에야 초기화 코드를 보게 된다면, 이미 변수가 어떤 맥락에서 선언되었는지 잊어버릴지도 모르니까요.  

변수와 변수에 값을 채우는 코드들의 순서를 이전 그대로 둘 수는 없습니다.  

데이터 종속을 수작업으로 분석한다면, 결국 실수를 하게 될 것입니다.  
구조만 개선하려고 하다가 실수로 동작까지 변경하게 될 수도 있습니다.  
그래도 문제는 없습니다.  
올바른 버전의 코드로 되돌리면 됩니다.  
작은 단계로 작업하세요.  

> 변수 선언과 초기화 위치가 떨어져있는건 그나마 나음  
> 해당 변수가 초기화 되었음에도 값을 계속 덮어씌우는 스타일이 문제라고 생각함  
> 변수 선언과 초기화를 분리해야한다면 최소한 final을 선언해야한다고 생각함  
>
> 코틀린 변수 선언과 초기화별 동작 방식
>
> 1. val, var 변수를 선언만 하는 경우 빌드 가능  
> val number: Int  
> var number: Int  
>
> 2. val, var 변수를 선언하고, 사용하는 경우 빌드 불가능  
> val number: Int  
> var number: Int  
> println(number)  
>
> 3. lateinit var 변수를 선언하고, 사용하는 경우 빌드 가능, UninitializedPropertyAccessException 발생    
> lateinit var text: String  
> println(text)  
> 
> 4. Delegates val, var 변수를 선언하고, 사용하는 경우 빌드 가능, IllegalStateException 발생  
> val number by Delegates.notNull<Int>()  
> var number by Delegates.notNull<Int>()  
> println(number)   
