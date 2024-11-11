# Chaining
* 코드를 방금 정리했는데 또 정리해도 될까?
  * 이것을 관리하는 것도 코드 정리의 핵심 기술이다
* 아주 작은 단계로 나누어 코드를 정리하자
### Guard clause
* 보호 구문으로 정리하면, 조건이 도우미로 드러나거나 설명하는변수로 추출되는 혜택을 얻는다
### Dead code
* 지우자, 응집도를 높이는 배치가 보인다
### Normalize symmetries
* 대칭으로 맞추다 보면, 유사한 코드들이 묶여진 순서대로 읽을 수 있다
### New interface, old implementation
* 새 인터페이스를 만들면 바로 사용하고 싶지만, 일일이 변환해 줘야 한다. 팬아웃 현상이 발생
  * 하나를 정리하면 한 다발의 정리가 이어지고 거기서 또... 연쇄적으로 이어지게 된다
### Reading order
* 읽는 순서를 정리하면, 대칭으로 맞출 기회가 생길 수 있다
### Cohesion order
* 응집도를 높이는 배치로 묶인 요소는 하위 요소로 추출할 후보가 된다
* Helper를 만드는 것은 코드 정리를 넘어서지만 익숙해지고 자시감이 생기면 더 큰 규모로 변경할 수 있다
### Explaining variables
* 변수 이름으로 알 수 있는 설명 덕분에 불필요한 주석이 드러나 삭제할 수 있다
### Explaining constants
* 설명하는 상수는 응집도를 높이는 배치를 이끈다, 한 번에 바뀌는 상수를 모아서 묵어 놓으면 나중에 바꾸기 좋다
### Explicit parameters
* 매개변수를 명확하게 만들면 집합으로 묶어 객체로 만들고 코드를 옮길 수 있다.
### Chunk statements
* 코드 덩어리에 주석을 붙이거나 Helper로 바꿀 수 있다
### Extract helper
* Helper 추출 이후에 보호 구문을 도입하거나, 설명하는 상수, 변수를 추출하거나, 주석을 지울 수도 있다
### One pile
* 비슷한 코드끼리 정리, 설명하는 주석 정리, Helper 추출을 기대할 수 있다
### Explaining comments
* 다른 방법(설명하는 변수, 상수, Helper) 등으로 도입할 수 있다면 주석에 있는 정보를 코드로 옮긴다
### Delete redundant comments
* 읽는 순서를 개선하고, 명시적인 매개변수를 쓰는 기회가 생긴다

## Conclusion
* 대대적으로 바꾸려고 하지 않게 주의하자
* 작은 정리를 순차적으로 성공하는 것이 무리한 정리로 실패하는 것보다 시간을 아껴준다