# Extract Helper
* 바꾸려는 부분을 helper 함수로 만들자
  * 테스트도 더 쉬워지는 장점이 있다

```java
public void changeUserStatus(int userId, Status targetStatus) {
    User user = userRepository.findById(userId);
    
    if(userStatus.equals(Status.WITHDRAWAL)) {
        throw new UserStatusTransitionFailedException(user.stats, targetStatus);
    }
    
    if(userStatus.equals(Status.MIGRATING) && targetStatus.equals(Status.NORMAL)) {
        user.status = Status.BLOCKED;
    }
    
    user.status = targetStatus;
    
    userRepository.update(user);
}
```

* 좋은 예제는 아니지만 (userStatus의 transite에 대한 validation은 user 스스로 하는게 더 좋아보임)
* 아래와 같이 helper로 추출하여 해당 동작만을 테스트하거나 변경할 수 있다.

```java
public void changeUserStatus(int userId, Status targetStatus) {
    User user = userRepository.findById(userId);
    
    userStatusTransitionValidator(user, targetStatus);
    
    userRepository.update(user);
}

public void userStatusTransitionValidator(User, Status targetStatus) {
    if(userStatus.equals(Status.WITHDRAWAL)) {
    throw new UserStatusTransitionFailedException(user.stats, targetStatus);
    }

    if(userStatus.equals(Status.MIGRATING) && targetStatus.equals(Status.NORMAL)) {
    user.status = Status.BLOCKED;
    }

    user.status = targetStatus;
}
        
```