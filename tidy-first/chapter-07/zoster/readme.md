# Move Declaration and Initialization Together
```java
public Float calculateExchangeRate(int itemId, int countryId) {
        Float exchangeRate;

        Float price;

        price = priceRepository.getByItemId(itemId);
        if(price == null) {
            throw new PriceDoesntExistsExcetpion(itemId);
        }

        exchangeRate = exchangeRateRepository.getByCountryId(countryId);
        if(exchangeRate == null) {
            exchangeRate = 1_000f;
        }

        price = price * exchangeRate;

        return price;
}
```

* 선언과 초기화를 같이 하자.

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