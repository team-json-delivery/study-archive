# 1. 보호 구문 (guard clause)
- 아래와 같이 중첩 코드가 생기면 가독성이 매우 떨어짐
```
if(조건)
    if(다른 조건 부정)
        .... 코드.....
```
- 이럴땐 아래와 같이 정리하자
```
if(조건 부정) return 
else(다른 조건) return 
.... 코드.....
```
- 다만 보호 구문을 남용하지 말자 -> 읽기 매우 까다로워짐

<img src="./img.png" alt="drawing" width="700px"/><br>
