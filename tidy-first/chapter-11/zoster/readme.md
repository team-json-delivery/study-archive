# Chunk Statements
```java
public class UserService {
    private static final String URL = "jdbc:mysql://localhost:3306/your_database";
    private static final String USERNAME = "your_username";
    private static final String PASSWORD = "your_password";    
    public void updateUserStatus(String userId, String status) {
        String sql = "UPDATE users SET status = ? WHERE user_id = ?";
        try (Connection conn = DriverManager.getConnection(URL, USERNAME, PASSWORD);
            PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setString(1, status);
            pstmt.setString(2, userId);
            int affectedRows = pstmt.executeUpdate();
            System.out.println(affectedRows + " rows updated.");
        } catch (SQLException e) {
            e.printStackTrace();
            System.out.println("Error updating user status: " + e.getMessage());
        }
    }
}
```

* 코드를 읽기 좋게 블럭 단위로 나누자.

```java
public class UserService {
    
    private static final String URL = "jdbc:mysql://localhost:3306/your_database";
    private static final String USERNAME = "your_username";
    private static final String PASSWORD = "your_password";
    
    public void updateUserStatus(String userId, String status) {
        String sql = "UPDATE users SET status = ? WHERE user_id = ?";
        
        try (Connection conn = DriverManager.getConnection(URL, USERNAME, PASSWORD);             
            PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setString(1, status);
            pstmt.setString(2, userId);
            
            int affectedRows = pstmt.executeUpdate();
            System.out.println(affectedRows + " rows updated.");
            
        } catch (SQLException e) {
            e.printStackTrace();
            System.out.println("Error updating user status: " + e.getMessage());
        }
    }
}
```