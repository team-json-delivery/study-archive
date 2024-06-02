# 03. 대칭으로 맞추기

- 아직 계산하지 않았다면 foo의 값을 계산하고 임시 보관 하라. 라는 의미라면,
```
foo()
  return foo if foo not nil
  foo := ...
  return foo
  
foo()
  if foo is nil
    foo := ...
  return foo

foo()
  return foo not nul
    ? foo
    : foo := ...

foo()
  return foo := foo not null
    ? foo
    : ...

foo()
  return foo := foo || ...
```

- 코드 스타일을 한가지 방식을 선택해서 코딩하자.

---
> 레거시에서도 적용이 가능 할까?