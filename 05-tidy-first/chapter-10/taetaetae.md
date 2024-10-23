# 10. 명시적인 매개변수
- 객체를 넘길때 명시적으로 사용하는 값을 넘기고 싶을 상황에서는
- 앞부분에서 매개변수 값을 채운 후, 뒷부분에서 명시적으로 전달
- e.g. map
- as-is
```java
params = { a:1, b:2}
foo(params)

function foo(params)
    ...params.a 
    ...params.b
```

- to-be
```java
function foo(params)
  foo_body(params.a, params.b)

function foo_body(a, b)
    ...a
    ...b
```

- 예전엔 foo 라는 메소드는 걷어내고 foo_body라는 메소드를 직접적으로 사용하는게 좋지 않을까 했는데
- 객체지향 책 보다보니 인터페이스와 메세지의 개념을 봤을때 객체 자체를 넘기는게 가독성에 좋다면 그게 올바를수도 있다고 하여 생각이 바뀌는중...