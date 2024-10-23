# Explicit Parameters
```java
public class UserDto {
    private String userId;
    private String username;
    private String email;
    private String phoneNumber;
    private String address;
    private String city;
    private String state;
    private String postalCode;
    private String country;
    private String occupation;
    private String company;
    private String department;
    private String title;
    private String gender;
    private String dateOfBirth;
    private List<String> hobbies;
    private boolean isVerified;
    private Date registrationDate;
    private String profilePictureUrl;
    private List<String> roles;
    private Map<String, String> preferences;
    private boolean agree;
}
```
```java
public Location getUserLocation(UserDto userDto) {
    int userId = userDto.userId;
    boolean userAgreed = userDto.agree;
    
    if(userAgreed) {
        return locationRepository.getUserLocation(userId);
    }
    
    ...
}
```
* 어떤 값이 필요한지 알 수 없는 함수는 고치기 어렵다.


```java
public Location caller(UserDto userDto) {
    return getUserLocation(userDto.userId, userDto.agree);
}

public Location getUserLocation(int userId, boolean userAgreed) {
    if(userAgreed) {
    return locationRepository.getUserLocation(userId);
    }

    ...
}
```