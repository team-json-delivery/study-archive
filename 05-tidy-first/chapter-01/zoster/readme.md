# Guard Clauses
* 아래와 같은 코드가 있다고 가정하자.
```java
public int boundAdjust(int number) {
    if(number > 0) {
        if(number >= 10) {
            number = 10;
            return number;
        }
        number++;
    } else {
        number = 0;
    }

    return number;
}
```

* 분석해보니 아래와 같은 요구사항을 가진다.
```java
@Test
@DisplayName("10보다 크면 10으로 변환한다.")
void test1() {
    int actual = sut.boundAdjust(11);
    assertThat(actual).isEqualTo(10);
}

@Test
@DisplayName("0보다 작으면 0으로 변환한다.")
void test2() {
    int actual = sut.boundAdjust(-1);
    assertThat(actual).isEqualTo(0);
}

@Test
@DisplayName("0~9면 1을 더한다.")
void test3() {
    int actual = sut.boundAdjust(5);
    assertThat(actual).isEqualTo(6);
}
```

* 범위를 벗어나는 경우에 대한 보호 구문을 먼저 작성하고 주요 요구사항인 number를 증가시키는 코드를 작성한다.
```java
public int boundAdjust(int number) {
    if(number < 0) return 0;
    if(number >= 10) return 10;

    number++;

    return number;
}
```