# 01. 보호 구문

- as-is
```java
if (조건)
    ... 코드 ...
```

```java
if (조건)
  if (다른 조건 부정)
    ... 코드 ...
```

> 코드의 세부사항을 살펴보기 전에 염두에 두어야 할 몇가지 전제조건이 있다. 라고 말하는 것처럼 보임
- to-be
```java
if (조건 부정) return
if (다른 조건) return
    ... 코드 ...
```

- 읽기 쉬워진다.
- 예시 : https://github.com/Bogdanp/dramatiq/pull/470/files