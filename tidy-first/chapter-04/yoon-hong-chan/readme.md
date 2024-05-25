# 4. 새로운 인터페이스로 기존 루틴 부르기
- 루틴을 호출해야하는데, 기존 인터페이스 때문에 어렵거나, 복잡하거나, 혼란스러울땐 호출하고 싶은 인터페이스를 새롭게 구현해서 호출
- 새로 만든 인터페이스는 기존 인터페이스를 호출하는 것으로 구현 가능
- 새롭게 구현한 통로 인터페이스는 소프트웨어 설계에서 작은 중추적인 역할 -> 어떤 동작 변경 시, 통로 인터페이스만 변경
### 이해 포인트
- 기존 인터페이스와 구현
```
interface OldService {
    void performOldTask(String data, int code);
}

class OldServiceImpl implements OldService {
    @Override
    public void performOldTask(String data, int code) {
        System.out.println("Old Service Implementation: " + data + ", " + code);
    }
}
```
- 새로운 인터페이스와 이를 통해 기존 인터페이스 호출하기
```
interface NewService {
    void performTask(String data);
}

class NewServiceImpl implements NewService {
    private final OldService oldService;

    public NewServiceImpl(OldService oldService) {
        this.oldService = oldService;
    }

    @Override
    public void performTask(String data) {
        // 새로운 인터페이스를 통해 기존 인터페이스 호출
        int defaultCode = 100;  // 예를 들어, 기본값 설정
        oldService.performOldTask(data, defaultCode);
    }
}
```
- 활용 예시
```
public class Main {
    public static void main(String[] args) {
        OldService oldService = new OldServiceImpl();
        NewService newService = new NewServiceImpl(oldService);

        // 새로운 인터페이스 사용
        newService.performTask("Hello, World!");
    }
}
```
- 인터페이스를 쓰지 않으면 main 함수에 oldService에 적용되어야할 로직(default 값)이 포함<br>
  -> 추후 다른 호출 부분에도 해당 로직(default 값)에 복사/붙여 넣기 또는 로직 변경 시 수정을 해야하는 좋지 않은 상황 발생(SRP 위반)
