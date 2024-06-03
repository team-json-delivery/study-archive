# Explaining Constants

```java
public Float calculateExchangeRate(int itemId, int countryId) {
        Float price = priceRepository.getByItemId(itemId);
        if(price == null) {
            throw new PriceDoesntExistsExcetpion(itemId);
        }

        Float exchangeRate = exchangeRateRepository.getByCountryId(countryId);
        if(exchangeRate == null) {
            exchangeRate = 1_000f;
        }
        price = price * exchangeRate;

        return price;
}
```

* 1_000f 는 뭘 의미하는지 알 수가 없다. 설명하는 상수로 바꾸자.

```java
public static final Float BASE_RATE = 1_000f;
...
        Float exchangeRate = exchangeRateRepository.getByCountryId(countryId);
        if(exchangeRate == null) {
            exchangeRate = BASE_RATE;
        }
        price = price * exchangeRate;

...
```