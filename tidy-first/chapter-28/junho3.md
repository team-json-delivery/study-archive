## Chapter 28. 되돌릴 수 있는 구조 변경

선행 코드 정리의 특징은 구조 변경은 대체로 되돌릴 수 있다는 점입니다.  

반면에, 후회할 만한 동작 변경과 대조해 보세요.  
모르긴 해도 해결하는 데 큰 비용이 들겁니다.  

일반적으로 되돌릴 수 있는 결정은 되돌릴 수 없는 결정과 다르게 취급해야 합니다.  
되돌릴 수 없는 결정에 대해서는 면밀히 검토하고 두 번, 세 번 확인하는 일은 매우 가치 있는 일입니다.  

대부분의 소프트웨어 설계 결정은 쉽게 되돌릴 수 있습니다.  
동작 변경을 쉽게 할 수 있는 장점이 있습니다.  
하지만, 부작용은 크지 않습니다.  
잘못된 결정이었다고 깨달으면, 쉽게 되돌릴 수 있으니까요.  

따라서, 실수를 피하는 것이 신경 쓸 정도로 큰일이 아니기 때문에 지나치게 노력할 필요가 없습니다.  

코드 검토 절차는 되돌릴 수 있는 변경 사항과 되돌릴 수 없는 변경 사항을 구분하지 않습니다.  
결국 너무나 다른 결과를 초래하는 일에 똑같이 시간 투자를 하는 꼴입니다.  

되돌릴 수 없는 설계 변경은 어떻게 하면 좋을까요?  
예를 들어, '서비스로 추출하기'는 비교적 큰 문제이기 때문에 되돌리기 어려울 수 있습니다.  
프로토타입을 실제로 구현해 볼 수도 있습니다.  
정리를 먼저 합시다.  
기능 플래그만 확ㅇ니하면 되니까요.  

'서비스로 추출하기'를 적어도 당분간은 되돌릴 수 있게 만들고 있습니다.  

되돌릴 수 있었던 설계 결정이 되돌릴 수 없게 되는, 또 다른 시나리오는 설계 결정이 코드 베이스 전체에 전하될 때입니다.  

> 카나리 배포  
> 슬픈 사실은 트래픽이 많지 않은 서비스(파드 2개)에서 카나리 배포는 큰 의미가 없다는 점  
> 물론, 임시로 전체 파드를 늘릴 수도 있겠지만.  
> 
> 매일 배포하는 팀이 되는 여정(2) — Feature Toggle 활용하기  
> https://medium.com/daangn/%EB%A7%A4%EC%9D%BC-%EB%B0%B0%ED%8F%AC%ED%95%98%EB%8A%94-%ED%8C%80%EC%9D%B4-%EB%90%98%EB%8A%94-%EC%97%AC%EC%A0%95-2-feature-toggle-%ED%99%9C%EC%9A%A9%ED%95%98%EA%B8%B0-b52c4a1810cd  
