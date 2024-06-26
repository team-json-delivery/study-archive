# 21. 코드 정리 시점
- 시스템의 동작 변경과 연관된 코드 정리 시점
### 아예 안한다면
- 미래에도 코드 동작이 변경되지 않을꺼면 코드 정리할 하지 않아도 된다는 말이 합리적일수도 있음
### 나중에 정리하기
- 날던 새가 물리 법칙에 의문을 품고, 갑자기 하늘에서 떨어지는 것과 같은 이야기 -> 언제나 일은 많기에 나중에 정리하기는 힘듬
- 아래와 같은 다른 방식으로 가치 창출
  - 지저분함 보유세를 줄여주는 것
  - 학습 도구로 활용하는 것 -> 설계한 세부 결과를 깨달을 수 있는 좋은 방법
  - 자신이 원할 때하면, 더 의욕적이고 기분도 좋음
### 동작 변경 후에 코드 정리
- 아래와 같은 경우 동작 변경 후 코드 정리
  - 방금 고친 코드를 다시 변경할 예정일 때
  - 지금 정리하는 것이 더 저렴할 때
  - 코드 정리하는 데 드는 시간이 동작 변경에 드는 시간과 거의 비슷할 때
### 코드 정리 후에 동작 변경
- 코드 정리는 선행해야하는 건가? 상황에 따라 다르기에 아래와 같은 질문을 해봐야함
  - 지저분한 상태로 코드 변경하면 일이 더 어려운지 -> 코드를 정리한다고 쉬워지지 않으면 먼저 정리 필요 없음
  - 코드 정리의 이점을 바로 얻을 수 있는지? -> 코드 정리한다면 더 빨리 이해가 되면 먼저 코드 정리
  - 코드 정리에 드는 비용을 보상 받기 좋을때 -> 이 코드를 변경하면 몇년동안 매주 보상 받음
  - 코드 정리에 대해 얼마나 확신이 드는지? -> 추측할 때는 편견에서 벗어나야함
- 코드를 먼저 정리하는 것도 좋지만, 정리 자체가 목적이 되지 않도록 경계 필요
### 요약
- 코드 정리 X
  - 미래 코드 변경 없을 때
  - 설계를 개선하더라도 배울 것이 없을 때
- 나중으로 미루자
  - 정리할 코드 분량이 많은데, 보상이 바로 보이지 않을 때
  - 코드 정리에 대한 보상이 잠재적일 때
  - 작은 묶음으로 여러 번에 나눠서 코드 정리를 할 수 있을 때
- 동작 변경 후에 정리
  - 다음 코드 정리까지 기다릴수록 비용이 더 불어날 때
  - 코드 정리를 하지 않으면 일을 끝냈다는 느낌이 들지 않을 때
- 코드 정리 후 동작 변경
  - 코드 정리가 동작변경에 즉각적인 효과가 있을 때
  - 어떤 코드를 어떻게 정리해야하는지 알고 있을 때