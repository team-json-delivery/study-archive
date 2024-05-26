# Normalize Symmetries


* currency가 null이면 exchangeRate을 반환하고 아니면 currency * exchangeRate을 반환하는 함수가 있다고 하자.
```java
public float calculateExchangeRate1(Integer currency, float exchangeRate) {
    return currency != null ? currency * exchangeRate : 1 * exchangeRate;
}

public float calculateExchangeRate2(Integer currency, float exchangeRate) {
    if(currency != null) {
        return currency * exchangeRate;
    }
    return 1 * exchangeRate;
}

public float calculateExchangeRate3(Integer currency, float exchangeRate) {
    return (currency != null ? currency : 1) * exchangeRate;
}

public float calculateExchangeRate4(Optional<Integer> currencyOpt, float exchangeRate) {
    return currencyOpt.orElse(1) * exchangeRate;
}
```

* 의미는 모두 동일하지만 여러가지 방법으로 구현할 수 있다.
* 중요한 것은 이와 같이 null일때 어떤 동작을 하는 코드를 짜고자 한다면 한가지 패턴을 선택하고 정해야 한다.
* 코드를 읽기 쉬워지고 `예측`이 가능해진다. 