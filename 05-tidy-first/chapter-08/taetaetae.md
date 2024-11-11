# 08. 설명하는 변수
- as-is
    ```java
    return new Point(
      ...긴 표현식...,
      ...다른ㄹ 긴 표현식...
    )
    ```

- to-be
    ```java
    x := ...긴 표현식...
    y := ...다른 긴 표현식...
    return new Point(x, y)
    )
    ```
- 힘들에 파악한 내용을 다시 코드에 넣는다.
  - 표현식과 분리되었기 때문에 코드 변경시 둘중 하나만 읽으면 됨
- 코드 정리에 대한 커밋과 동작 변경에 대한 커밋은 분리해야 한다.