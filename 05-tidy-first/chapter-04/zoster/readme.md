# New interface, Old Implementation
```JAVA
interface A {
    int getPostalCode(int cityNumber, int roadNumber, int countryCode);
}

class AImpl {
    
    Repository respository;
    
    public int getPostalCode(int cityNumber, int roadNumber, int countryCode) {        
        return repository.get(countryCode, cityNumber, roadNumber);
    }
}
```
* Global service를 준비하며 야심차게 country code까지 포함된 interface를 만들었지만 한국만 계속 서비스하고 있는 코드

```java
class Use {
    AImpl aimpl;
    public getPostalCode(int cityNumber, int roadNumber) {
        return repository.get(410, cityNumber, roadNumber);
    }
}
```

* 그냥 하나 새로 만들고 제거하자
```JAVA
interface A {
    int getPostalCode(int cityNumber, int roadNumber, int countryCode);
    int getKoreaPostalCode(int cityNumber, int roadNumber);
}

class AImpl {
    
    Repository respository;

    public int getPostalCode(int cityNumber, int roadNumber, int countryCode) {
        return repository.get(countryCode, cityNumber, roadNumber);
    }
    
    public int getKoreaPostalCode(int cityNumber, int roadNumber) {
        return getPostalCode(410, cityNumber, roadNumber);
    }
}
```
* to-be

```JAVA
interface A {
    int getKoreaPostalCode(int cityNumber, int roadNumber);
}

class AImpl {
    
    Repository respository;

    public int getKoreaPostalCode(int cityNumber, int roadNumber) {
        return repository.get(410, cityNumber, roadNumber);
    }
}
```

* 거꾸로 코딩, TDD, Helper 사용도 모두 비슷한 효과를 누릴 수 있는데 필요한 것을 먼저 만든다.(interface를 만든다와 같은 개념)
* 이후 그 필요한 것들만을 채우기 위한 코드를 작성하면 구조화와 동시에 YAGNI(You Aren't Gonna Need It)도 지킬 수 있다.