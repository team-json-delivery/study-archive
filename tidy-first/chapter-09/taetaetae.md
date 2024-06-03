# 09. 설명하는 상수
- as-is
    ```java
    if reesponse.code = 404
        ...
    ```
- to-be
    ```java
    PAGE_NOT_FOUND := 404
    if reesponse.code = 404
      ...
    ```
- 같은 리터럴 상수가 두곳에서 나타날때는 다른의미로 사용되는지 확인
  - 이때는 같은 값이라도 분리해서 정의해야 하지 않을까
- 만약 상수 문쟈열이 문자열 자체에서 그 의미를 담고 있으면 구지 상수로 안뽑아도 되지 않을까?
    ```java
    e.g.
    NOT_FOUND := "NOT_FOUND"
    if response.message = NOT_FOUND
        ...
    
    ```