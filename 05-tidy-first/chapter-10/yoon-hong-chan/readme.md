# 10. 명시적인 매개변수
- 루틴에서 다루는 일부 데이터가 명시적으로 전달되지 않으면? 루틴을 나누자. 앞부분에서 매개변수 값을 채운 후, 뒷부분에서 명시적으로 전달
- 아래와 같이 매개변수를 map을 사용하여 어떤 데이터가 필요한지 알기가 어려움 -> 끔찍한 남용의 길이 열림
```
params = { a: 1, b: 2}
foo(params)

function foo(params)
    ...params.a... ...params.b...
```
- 아래와 같이 foo를 나누면 매개변수 정리가 가능
```
function foo(params)
    foo_body(params.a, params.b)
function foo_body(a,b)
    ...a... ...b...
```
### 이해 포인트
- 매개변수를 명시하기 위해 기존 foo 로직에서 foo_body로 wrapping 함