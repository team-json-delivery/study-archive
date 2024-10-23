# Separate Tidying
* PR / Code review를 사용한다고 가정하자
1. 동작 변경과 Refactoring 코드를 같이 넣는다
2. Reviewer들이 PR이 너무 길다고 불평한다
3. Refactoring과 동작 변경 PR을 분리한다
4. Reviewer들이 Refactoring만 하는 PR이 무슨 의민지 모르겠다고 불평한다 (?)
5. 1로 돌아간다
* 정리하면 Refactoring는 별도의 PR로 만들고 PR당 소규모의 Refactoring만 포함한다
* 일반적으로 Refactoring를 배우는 사람들은 예상되는 단계를 거치는데
  1. 변경을 구분하지 않은 상태에서 다수의 변경을 반영하기 시작한다
  2. 변경 중간에 구조 변경이 필요함을 인지하고 구조를 변경한다
  3. 공통 흐름을 알아채고 정리 / 변경을 반복한다
  4. 거대한 하나의 PR에 이러한 반복 과정들이 포함된다
  5. 이것은 Reviewer들을 주저하게 만들기 때문에 변경을 나누어 별도의 PR로 만들어야 한다
* 이것은 trade-off가 있지만 작은 PR은 일반적으로 review 시간 단축으로 환영 받는다
  * 다만 그럼에도 review가 느리다면 더 큰 PR을 만들게 되어 이후 review를 더 느리게 만든다
* 이후 이러한 작업에 숙달되면 Refactoring PR은 review 대상에서 제외하는 것도 고려해볼만하다(!?)