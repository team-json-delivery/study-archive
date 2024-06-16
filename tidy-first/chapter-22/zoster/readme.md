# Beneficially Relating Elements
* 소프트웨어의 각 요소들은 서로 관계를 맺고 있다.(함수가 다른 함수를 호출 한다던가)
* 전역 네임스페이스를 사용하여 어셈블리어로 프로그램을 만들어두면 외부에서 봤을때는 다른 프로그램과 동일하게 동작하더라도 변경이 매우 어려울 수 있다
* 설계할 때 중간 요소를 만들어 이들간에 서로 도움을 주면 더 간단해질 수 있다(유익한 관계)
* 그렇다면 어떻게 유익한 관계를 만들 수 있을까?
* 예를들어 아래와 같은 코드가 있다
```java
int caller() {
    return box.width() * box.height();
}
```
* box 객체의 두개의 함수를 호출하는 관계를 가진다. 아래처럼 바꿔보자

```java
int caller() {
    return box.area();
}
class box {
    int width;
    int height;
    int area() {
        return width * height;
    }
}
```
* 객체에 두번 접근할 필요가 없을 뿐 아니라, area 내부에서 어떤 연산을 하는지 caller가 알 필요가 없게 의존성이 제거되었다.
