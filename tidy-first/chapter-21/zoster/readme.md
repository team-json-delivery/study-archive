# First, After, Later, Never
* 코드 정리는 언제 해야 할까?
* 아예 안한다면?
  * 앞으로 바뀌지 않을 코드라면 합리적일 수 있으나 그 외의 경우는 합리적이지 않다
  * 그리고 그런 코드는 예외적으로 아주 일부만 존재할 뿐이다
* 나중에 정리한다면?
  * 우리는 흔히 이건 결코 일어나지 않는 일이라고 말하곤 한다. (르블랑의 법칙)
  * 그러나 일할 시간이 충분하다면 어떻게 할 것인지 한번 생각해보자
  * 물론 먼저하면 좋지만, 어렵다면 해야할 엉망 목록을 정리해두자
  * 여기서 발생하는 가치가 있다
    * tax of messiness를 줄이는 것, legacy를 새 api로 바꾸는 과정을 보면 당장 영향 받는건 변경했지만 아직 변경을 반영할 곳이 남아있다. 이것을 미뤄둔 시간에 처리한다면 나중에 해야할 일들을 줄일 수 있다
    * (technical dept와 비슷한 의미로 쓰는 듯, K 개발용어인가?)
    * 두번째로는 학습 도구로 활용될 수 있다. 코드를 정리하다보면 무언가를 배우게 되고 또한 설계한 세부 결과를 깨달을 수 있는 좋은 방법이다
    * 마지막으로 나중에 자신이 원할 때 하면 더 의욕적이고 기분도 좋다!
* 동작 변경 후에 코드 정리
  * 일단 동작을 변경하고 보니 더 쉽게할 수 있는 방법을 알게 되었다. 코드를 정리해야 할까?
  * 상황에 따라 다르다. 또 변경해야 하는 코드라면(보통은 그렇다) 동작 변경 후 코드 정리는 상당히 의미가 있다
  * 또한 지금 맥락을 앍고 있을때 정리하는게 더 쉽다
  * 그럼 얼마나 해야 하는가? 동작 변경에 투자한 시간 만큼 정도가 적절하지 않을까?
* 코드 정리 후에 동작 변경
  * 정리한다고 더 쉬워지지 않는다면 먼저 정리하지 말자
  * 정리의 이점을 바로 얻을 수 있지 않다면 미루자
  * 정리에 드는 비용을 동작 변경의 효율 증가와 경제적으로 비교하여 수행하는 것이 좋다
  * 정리에 확신이 없다면 정리하지 말자
  * 정리 그 자체가 목적이 되면 안된다
