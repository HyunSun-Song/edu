---
typora-root-url: ./images
typora-copy-images-to: ./images
---

**시나리오 1**

**상황:** 통신사 K는 핸드폰 구매 처리 과정에서 자주 오류가 발생하고 있습니다. 주요 문제는 아래와 같습니다:

1. 주문이 실패했지만 결제 처리가 됨
2. 주문이 성공했지만 배송 정보 저장이 실패해 배송이 불가능함

이러한 문제는 매출 손실과 고객 신뢰도 저하로 이어지고 있습니다. 따라서, 요금제 및 핸드폰 판매 과정의 신뢰성을 개선해야 합니다.

**문제:**

1. 배송 정보 저장이 실패한 경우 주문 성공을 막는 방안
   **힌트**: 트랜잭션(transaction)을 사용하여 여러 데이터베이스 작업을 하나의 단위로 묶는 방법을 고려하세요. 트랜잭션이 실패하면 모든 작업이 롤백(rollback)되어 데이터베이스 상태가 일관되게 유지됩니다.
2. 주문 실패한 경우, 외부 시스템 결제 정보를 자동 취소처리 하는 방안
   **힌트**: 트랜젝션이 실패하면 보상처리 방법을 고려하세요..

<br>**통신사 K가 핸드폰 구매 처리 과정에서 발생하는 문제를 해결하기 위해 트랜잭션 및 보상 처리 방법을 적용하는 방안을 다음과 같이 구성할 수 있습니다.**

### 1. 배송 정보 저장이 실패한 경우 주문 성공을 막는 방안

주문 처리 과정에서 여러 작업이 성공적으로 완료되어야 전체 주문이 성공으로 간주되도록 하려면, **트랜잭션**을 활용하여 여러 데이터베이스 작업을 하나의 단위로 묶어야 합니다. 트랜잭션은 모든 작업이 성공적으로 완료되거나, 하나라도 실패할 경우 모든 작업이 롤백되어 데이터베이스 상태를 일관되게 유지합니다.

#### **트랜잭션을 이용한 데이터베이스 작업**

- **트랜잭션 시작**: 주문 처리의 시작 지점에서 트랜잭션을 시작합니다.
- **주문 정보 저장**: 주문 정보를 데이터베이스에 저장합니다.
- **결제 처리**: 결제 정보가 성공적으로 처리됩니다.
- **배송 정보 저장**: 배송 정보를 데이터베이스에 저장합니다.
- **트랜잭션 커밋**: 모든 작업이 성공적으로 완료된 경우 트랜잭션을 커밋하여 변경 사항을 확정합니다.
- **트랜잭션 롤백**: 어느 하나라도 실패할 경우, 트랜잭션을 롤백하여 모든 작업을 취소하고 데이터베이스 상태를 원상태로 복원합니다.

**예시: SQL 트랜잭션**

```
sql코드 복사BEGIN TRANSACTION;

-- 1. 주문 정보 저장
INSERT INTO Orders (OrderID, CustomerID, ProductID, Quantity) VALUES (1, 123, 456, 2);

-- 2. 결제 처리
INSERT INTO Payments (OrderID, PaymentAmount) VALUES (1, 200);

-- 3. 배송 정보 저장
INSERT INTO Shipping (OrderID, Address, ShippingDate) VALUES (1, '123 Main St', '2024-08-09');

-- 모든 작업이 성공하면 커밋
COMMIT;

-- 오류 발생 시 롤백
ROLLBACK;
```

**예시: Java (JDBC) 트랜잭션**

```
java코드 복사import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;

public class OrderProcessor {
    public void processOrder(int orderId, int customerId, int productId, int quantity, double paymentAmount, String address) {
        Connection conn = null;
        try {
            conn = DriverManager.getConnection("jdbc:mysql://localhost/dbname", "user", "password");
            conn.setAutoCommit(false); // 트랜잭션 시작
            
            Statement stmt = conn.createStatement();
            
            // 1. 주문 정보 저장
            stmt.executeUpdate("INSERT INTO Orders (OrderID, CustomerID, ProductID, Quantity) VALUES (" + orderId + ", " + customerId + ", " + productId + ", " + quantity + ")");
            
            // 2. 결제 처리
            stmt.executeUpdate("INSERT INTO Payments (OrderID, PaymentAmount) VALUES (" + orderId + ", " + paymentAmount + ")");
            
            // 3. 배송 정보 저장
            stmt.executeUpdate("INSERT INTO Shipping (OrderID, Address) VALUES (" + orderId + ", '" + address + "')");
            
            // 모든 작업이 성공하면 커밋
            conn.commit();
        } catch (SQLException e) {
            if (conn != null) {
                try {
                    conn.rollback(); // 오류 발생 시 롤백
                } catch (SQLException ex) {
                    ex.printStackTrace();
                }
            }
            e.printStackTrace();
        } finally {
            if (conn != null) {
                try {
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

### 2. 주문 실패 시 외부 시스템 결제 정보를 자동 취소 처리

주문 처리 중 오류가 발생하면 외부 결제 시스템에서 결제 정보를 자동으로 취소하는 보상 처리 방법을 적용해야 합니다. 이 경우, **보상 트랜잭션**을 사용하여 결제 취소와 같은 반대 작업을 수행합니다.

#### **보상 트랜잭션을 통한 결제 취소**

- **오류 발생 시 결제 취소 요청**: 주문 처리 중 오류가 발생하면 결제 취소 요청을 외부 결제 시스템에 자동으로 전송하여 결제를 취소합니다.
- **결제 취소 확인**: 결제 취소 요청이 성공적으로 처리되었는지 확인하고, 오류가 발생할 경우 추가적인 조치를 취합니다.

**예시: 결제 취소 API 호출**

```
java코드 복사import java.io.IOException;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.URL;

public class PaymentService {
    public void cancelPayment(int orderId) throws IOException {
        URL url = new URL("https://payment-provider.com/api/cancel");
        HttpURLConnection connection = (HttpURLConnection) url.openConnection();
        connection.setRequestMethod("POST");
        connection.setRequestProperty("Content-Type", "application/json");
        connection.setDoOutput(true);

        String jsonInputString = "{ \"orderId\": " + orderId + " }";

        try (OutputStream os = connection.getOutputStream()) {
            byte[] input = jsonInputString.getBytes("utf-8");
            os.write(input, 0, input.length);
        }

        int responseCode = connection.getResponseCode();
        if (responseCode != HttpURLConnection.HTTP_OK) {
            throw new IOException("Failed to cancel payment: " + responseCode);
        }
    }
}
```

#### **보상 트랜잭션을 적용한 주문 처리 예시**

```
java코드 복사import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;

public class OrderProcessor {
    private PaymentService paymentService = new PaymentService();
    
    public void processOrder(int orderId, int customerId, int productId, int quantity, double paymentAmount, String address) {
        Connection conn = null;
        try {
            conn = DriverManager.getConnection("jdbc:mysql://localhost/dbname", "user", "password");
            conn.setAutoCommit(false); // 트랜잭션 시작
            
            Statement stmt = conn.createStatement();
            
            // 1. 주문 정보 저장
            stmt.executeUpdate("INSERT INTO Orders (OrderID, CustomerID, ProductID, Quantity) VALUES (" + orderId + ", " + customerId + ", " + productId + ", " + quantity + ")");
            
            // 2. 결제 처리
            stmt.executeUpdate("INSERT INTO Payments (OrderID, PaymentAmount) VALUES (" + orderId + ", " + paymentAmount + ")");
            
            // 3. 배송 정보 저장
            stmt.executeUpdate("INSERT INTO Shipping (OrderID, Address) VALUES (" + orderId + ", '" + address + "')");
            
            // 모든 작업이 성공하면 커밋
            conn.commit();
        } catch (SQLException e) {
            if (conn != null) {
                try {
                    conn.rollback(); // 오류 발생 시 롤백
                    paymentService.cancelPayment(orderId); // 결제 취소
                } catch (SQLException | IOException ex) {
                    ex.printStackTrace();
                }
            }
            e.printStackTrace();
        } finally {
            if (conn != null) {
                try {
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

### 결론

1. **배송 정보 저장 실패 시 주문 성공을 막기 위해** 트랜잭션을 사용하여 주문 처리 과정의 모든 단계를 하나의 단위로 묶어야 합니다. 모든 작업이 성공적으로 완료될 때만 커밋하며, 오류 발생 시 롤백하여 데이터 일관성을 유지합니다.
2. **주문 실패 시 결제 정보를 자동으로 취소하기 위해** 보상 트랜잭션을 사용하여 결제 취소 요청을 외부 시스템에 전송합니다. 이를 통해 결제와 주문의 일관성을 유지하고, 고객 신뢰도를 회복할 수 있습니다.

------



**시나리오 2**

**상황:** K 통신사는 고객의 개인정보를 안전하게 보호하기 위해 시스템 보안을 강화하려고 합니다. ABC 통신사는 고객의 비밀번호를 안전하게 저장해야 합니다. 이를 위해 다음과 같은 요구사항을 충족해야 합니다:

1. 비밀번호는 평문으로 저장되어서는 안 됩니다.
2. 해킹 공격(예: 무차별 대입 공격, 사전 공격)에 대비할 수 있는 충분한 보안 수준을 제공해야 합니다.

**문제:**

1. 고객의 비밀번호를 안전하게 저장하기 위해 어떤 방법을 사용할 수 있는가?
   **힌트**: 해시 함수(hash function)와 솔트(salt)를 사용하여 비밀번호를 안전하게 저장할 수 있습니다.
2. 고객이 안전하게 로그인할 수 있도록 로그인 절차를 설계하고, 이에 필요한 보안 대책을 설명하세요.
   힌트: JWT(JSON Web Token)와 같은 토큰 기반 인증을 사용하세요. TLS/SSL을 통해 데이터 전송 시 암호화하는 것도 중요한 보안 대책입니다.

 <br>K 통신사가 고객의 비밀번호를 안전하게 저장하고 로그인 절차를 강화하기 위해 아래와 같은 방법을 사용할 수 있습니다.

### 1. 고객의 비밀번호를 안전하게 저장하기 위한 방법

비밀번호를 평문으로 저장하지 않고 안전하게 보호하기 위해, **해시 함수**와 **솔트**를 사용하는 방법을 적용할 수 있습니다.

#### **비밀번호 해싱과 솔팅**

1. **해시 함수 사용**:
   - 해시 함수는 입력값을 고정된 길이의 문자열로 변환합니다. 비밀번호를 해시 함수에 입력하면, 원본 비밀번호와는 다른 해시값이 생성됩니다.
   - 암호화 해시 함수로는 `bcrypt`, `argon2`, `PBKDF2` 등이 권장됩니다. 이러한 함수들은 단방향 해시와 자동으로 솔트 기능을 제공하여 보안성을 높입니다.
2. **솔트(salt)**:
   - 솔트는 각 사용자마다 고유하게 생성되는 무작위 문자열입니다. 비밀번호 해시화 과정에서 솔트를 추가함으로써 동일한 비밀번호라도 서로 다른 해시값을 생성합니다.
   - 솔트는 해시 충돌 공격과 무차별 대입 공격을 방어하는 데 유용합니다.

#### **해시와 솔트를 사용한 비밀번호 저장 예시**

```
java코드 복사import org.mindrot.jbcrypt.BCrypt;

public class PasswordUtil {
    // 비밀번호를 해시화하여 저장
    public static String hashPassword(String plainPassword) {
        return BCrypt.hashpw(plainPassword, BCrypt.gensalt());
    }

    // 입력된 비밀번호와 저장된 해시 비밀번호를 비교
    public static boolean checkPassword(String plainPassword, String hashedPassword) {
        return BCrypt.checkpw(plainPassword, hashedPassword);
    }
}
```

**위 코드에서는** `bcrypt` 라이브러리를 사용하여 비밀번호를 해시화하고, 해시화된 비밀번호와 입력된 비밀번호를 비교합니다. `BCrypt.gensalt()`는 무작위 솔트를 생성하며, `BCrypt.checkpw()`는 입력된 비밀번호와 저장된 해시 비밀번호를 비교합니다.

### 2. 고객이 안전하게 로그인할 수 있도록 로그인 절차 설계

로그인 절차를 안전하게 설계하고 보안 대책을 적용하는 방법은 다음과 같습니다.

#### **로그인 절차**

1. **사용자 입력**:
   - 사용자로부터 이메일/사용자 이름과 비밀번호를 입력받습니다.
2. **비밀번호 검증**:
   - 입력된 비밀번호를 해시화하여 데이터베이스에 저장된 해시 비밀번호와 비교합니다.
3. **토큰 발급**:
   - 인증이 성공하면 JWT(JSON Web Token) 또는 다른 형태의 토큰을 생성하여 사용자에게 발급합니다.
4. **응답**:
   - 클라이언트에 인증된 세션을 유지하기 위해 토큰을 반환합니다.

#### **보안 대책**

1. **JWT (JSON Web Token)**:
   - JWT는 인증된 사용자에게 발급되는 토큰으로, 서버와 클라이언트 간의 상태를 유지하지 않고도 인증을 처리할 수 있습니다.
   - JWT는 페이로드(데이터), 헤더, 서명으로 구성되어 있으며, 서버는 비밀 키를 사용하여 서명을 검증합니다.
2. **TLS/SSL**:
   - 데이터 전송 시 TLS(Transport Layer Security) 또는 SSL(Secure Sockets Layer) 프로토콜을 사용하여 암호화된 채널을 제공합니다. 이를 통해 네트워크를 통한 데이터 전송 과정에서 중간자 공격 및 데이터 도청을 방지할 수 있습니다.

#### **로그인 절차 및 JWT 예시**

```
java코드 복사import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;

public class AuthenticationService {
    private static final String SECRET_KEY = "your-secret-key";

    // 사용자 로그인 처리
    public String loginUser(String username, String password) {
        // 비밀번호 검증
        String storedHashedPassword = getStoredPasswordHash(username);
        if (PasswordUtil.checkPassword(password, storedHashedPassword)) {
            // 비밀번호가 일치하면 JWT 생성
            return generateToken(username);
        } else {
            throw new SecurityException("Invalid username or password");
        }
    }

    // JWT 생성
    private String generateToken(String username) {
        return Jwts.builder()
                   .setSubject(username)
                   .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
                   .compact();
    }

    // JWT 검증
    public boolean validateToken(String token) {
        try {
            Jwts.parser()
                .setSigningKey(SECRET_KEY)
                .parseClaimsJws(token);
            return true;
        } catch (Exception e) {
            return false;
        }
    }

    // 사용자 비밀번호 해시 조회 (예시)
    private String getStoredPasswordHash(String username) {
        // 실제 구현에서는 데이터베이스에서 비밀번호 해시를 가져오는 과정이 필요합니다.
        return "$2a$10$uG9hB8.YaT/eOmtMKnQEX.w81A8w1xN0YQ0H.2gkW6cQi5FDmgmBe"; // 예시 해시
    }
}
```

**위 코드에서는** 사용자 인증이 성공하면 JWT를 생성하여 반환합니다. 클라이언트는 이 JWT를 사용하여 후속 요청에서 인증을 처리할 수 있습니다. `validateToken` 메서드는 JWT를 검증하여 유효성을 확인합니다.

### 결론

1. **비밀번호 안전하게 저장**:
   - 비밀번호를 평문으로 저장하지 말고, 해시 함수와 솔트를 사용하여 해시화합니다. `bcrypt`, `argon2`, `PBKDF2`와 같은 강력한 해시 알고리즘을 사용하여 비밀번호를 보호합니다.
2. **안전한 로그인 절차**:
   - 로그인 시 비밀번호 검증 후 JWT를 발급하여 세션 관리를 수행합니다. 데이터 전송 시 TLS/SSL을 사용하여 보안을 강화합니다.

<br>

------



**시나리오 3**

**상황:** K 통신사의 개발 팀은 사용자 정보 조회 기능을 포함한 예약 시스템을 개발 중입니다. 사용자가 자신의 정보를 조회하기 위해 사용자 ID를 입력하면, 서버는 다음과 같은 SQL 쿼리를 실행합니다:

```sql
SELECT * FROM users WHERE user_id = '<사용자 입력>';
```

여기서 <사용자 입력> 부분은 사용자가 웹 애플리케이션의 입력 필드에 직접 입력한 값으로 대체됩니다.

**문제:**

1. 위와 같은 입력이 취약한 이유를 설명하세요.
   **힌트**: SQL 인젝션(SQL Injection) 공격에 취약할 수 있습니다. 이는 사용자가 입력한 값이 그대로 SQL 쿼리에 포함되면서 발생하는 보안 취약점입니다.
2. 취약한 문제를 해결하기 위한 방안을 제시하세요.
   **힌트**: 준비된 문(Prepared Statements) 또는 ORM(Object-Relational Mapping)을 사용하여 사용자 입력 값을 안전하게 처리하세요.

<br>1. 위와 같은 입력이 취약한 이유

위의 SQL 쿼리는 다음과 같은 이유로 취약합니다:

**SQL 인젝션(SQL Injection) 공격에 취약함**

- 문제 설명

  :

  - 사용자가 입력한 값이 SQL 쿼리 문자열의 일부로 직접 포함됩니다. 이는 사용자가 악의적인 SQL 코드를 삽입할 수 있는 기회를 제공하므로, 데이터베이스에서 임의의 쿼리를 실행하거나, 데이터베이스의 구조를 변경하거나, 중요 정보를 탈취할 수 있습니다.
  - 예를 들어, 사용자가 입력 필드에 `'; DROP TABLE users; --`와 같은 값을 입력하면, 원래의 쿼리와 결합되어 데이터베이스에서 테이블을 삭제하는 쿼리가 실행될 수 있습니다.

**예시 공격**:

```
sql
코드 복사
SELECT * FROM users WHERE user_id = ''; DROP TABLE users; --';
```

이 쿼리는 `users` 테이블을 삭제하는 악성 쿼리를 실행하게 됩니다.

### 2. 취약한 문제를 해결하기 위한 방안

**SQL 인젝션 공격을 방지하기 위한 방법**:

1. 준비된 문(Prepared Statements) 사용

   :

   - 준비된 문을 사용하면 SQL 쿼리와 사용자 입력 값을 분리하여, 쿼리가 실행되기 전에 SQL 서버가 쿼리 구조를 미리 파싱합니다. 사용자 입력은 쿼리 구조와 분리되어 안전하게 처리됩니다.
   - 준비된 문을 사용하면 SQL 쿼리에서 사용자 입력이 쿼리의 구조를 변경하지 않도록 방지할 수 있습니다.

**예시 (Java JDBC)**:

```
java코드 복사import java.sql.*;

public class UserService {
    private static final String DB_URL = "jdbc:mysql://localhost:3306/mydatabase";
    private static final String USER = "username";
    private static final String PASS = "password";

    public User getUserById(String userId) throws SQLException {
        String query = "SELECT * FROM users WHERE user_id = ?";
        
        try (Connection conn = DriverManager.getConnection(DB_URL, USER, PASS);
             PreparedStatement stmt = conn.prepareStatement(query)) {
             
            stmt.setString(1, userId); // 사용자 입력을 안전하게 설정
            ResultSet rs = stmt.executeQuery();

            // 사용자 정보를 처리하는 로직
            if (rs.next()) {
                // 사용자 정보 추출
                // 예: return new User(rs.getString("user_id"), rs.getString("name"), ...);
            } else {
                // 사용자 정보가 없는 경우 처리
            }
        }
    }
}
```

**위 예시에서는** `PreparedStatement`를 사용하여 SQL 쿼리의 안전성을 보장합니다. 사용자의 입력값은 `stmt.setString()`을 통해 쿼리와 분리되어 안전하게 전달됩니다.

1. ORM (Object-Relational Mapping) 사용

   :

   - ORM 라이브러리(예: Hibernate, Entity Framework, Django ORM 등)를 사용하면, SQL 쿼리를 직접 작성하지 않고 객체 지향적으로 데이터베이스를 다룰 수 있습니다. ORM 라이브러리는 내부적으로 SQL 인젝션을 방지하는 기능을 제공하며, 개발자가 직접 SQL 쿼리를 작성하지 않기 때문에 보안성이 높습니다.

**예시 (Hibernate)**:

```
java코드 복사import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;

public class UserService {
    private static EntityManagerFactory emf = Persistence.createEntityManagerFactory("my-persistence-unit");

    public User getUserById(String userId) {
        EntityManager em = emf.createEntityManager();
        try {
            return em.find(User.class, userId); // 사용자 ID로 사용자 객체를 조회
        } finally {
            em.close();
        }
    }
}
```

**위 예시에서는** JPA(Hibernate)를 사용하여 SQL 쿼리를 직접 작성하지 않고도 안전하게 데이터베이스 접근을 수행합니다. `em.find()` 메서드는 자동으로 SQL 인젝션 공격을 방지합니다.

### 결론

1. **SQL 인젝션 취약점 설명**:
   - 사용자의 입력값이 SQL 쿼리의 일부로 직접 포함되면, 악의적인 SQL 코드가 삽입될 수 있으며 데이터베이스의 중요한 데이터가 손상되거나 탈취될 수 있습니다.
2. **취약점 해결 방안**:
   - **준비된 문(Prepared Statements)**: SQL 쿼리와 사용자 입력 값을 분리하여 안전하게 처리합니다.
   - **ORM(Object-Relational Mapping)**: ORM을 사용하면 SQL 쿼리를 직접 작성하지 않아도 되며, 내부적으로 SQL 인젝션 공격을 방지하는 기능을 제공합니다.

<br>

------



**시나리오 4**

**상황:** 통신사 K사는 고객의 통화 기록을 대량의 데이터로 관리하고 있습니다. 이 많은 데이터를 효율적으로 관리하고 쿼리 성능을 최적화하기 위해 테이블 개선을 해야 합니다. 통화 기록은 월별로 조회가 되며, 매달 1년이 지난 데이터는 삭제를 해야 합니다.

**통화 기록 테이블:**

```sql
CREATE TABLE call_records (
 id INT PRIMARY KEY,
 user_id INT,
 call_date DATE,
 duration INT,
 destination VARCHAR(50)
);
```

**문제:**

1. 월별 조회 성능 향상과 데이터 삭제를 효율적으로 하기 위한 방안을 제시하세요.
   **힌트**: 파티셔닝(partitioning)을 사용하여 데이터를 월별로 분할하세요.
2. 위에 제시한 방안의 장점을 설명하세요.
3. 데이터 삭제를 자동화하는 방안을 제시하세요.
   **힌트**: 스케줄러나 이벤트 트리거를 사용하여 일정 기간마다 자동으로 오래된 데이터를 삭제하는 스크립트를 실행할 수 있습니다.

<br>1. 월별 조회 성능 향상과 데이터 삭제를 효율적으로 하기 위한 방안

**파티셔닝(Partitioning) 사용**

- **테이블 파티셔닝**: 테이블을 파티셔닝하여 데이터를 월별로 분할합니다. 이를 통해 특정 기간의 데이터를 조회할 때 전체 테이블을 검색할 필요 없이 해당 월의 데이터만 조회하게 되며, 데이터 삭제도 효율적으로 수행할 수 있습니다.

**예시: 월별 파티셔닝을 사용한 테이블 정의**

```
sql코드 복사CREATE TABLE call_records (
    id INT,
    user_id INT,
    call_date DATE,
    duration INT,
    destination VARCHAR(50),
    PRIMARY KEY (id, call_date)
)
PARTITION BY RANGE (YEAR(call_date)) (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION p2026 VALUES LESS THAN (2027)
);
```

- **범위 파티셔닝**: `call_date` 컬럼을 기준으로 범위 파티셔닝을 사용합니다. 위 예시에서는 연도를 기준으로 데이터를 나누고 있습니다. 월별 파티셔닝이 필요한 경우 `YEAR(call_date)` 대신 `YEAR(call_date) * 100 + MONTH(call_date)`과 같은 계산식을 사용할 수 있습니다.

**월별 파티셔닝 예시**

```
sql코드 복사CREATE TABLE call_records (
    id INT,
    user_id INT,
    call_date DATE,
    duration INT,
    destination VARCHAR(50),
    PRIMARY KEY (id, call_date)
)
PARTITION BY RANGE (TO_DAYS(call_date)) (
    PARTITION p202301 VALUES LESS THAN (TO_DAYS('2023-02-01')),
    PARTITION p202302 VALUES LESS THAN (TO_DAYS('2023-03-01')),
    PARTITION p202303 VALUES LESS THAN (TO_DAYS('2023-04-01')),
    -- 추가 파티션 정의
    PARTITION p202312 VALUES LESS THAN (TO_DAYS('2024-01-01'))
);
```

### 2. 위에 제시한 방안의 장점 설명

**장점**:

- **성능 향상**:
  - **빠른 조회**: 파티셔닝된 테이블은 특정 파티션만 스캔하면 되므로 월별 데이터 조회 시 전체 테이블을 검색할 필요가 없어 쿼리 성능이 향상됩니다.
  - **효율적인 인덱스 사용**: 각 파티션은 독립적으로 인덱스를 관리할 수 있어 인덱스 탐색 성능이 향상됩니다.
- **데이터 관리 용이**:
  - **효율적인 삭제**: 오래된 데이터를 삭제할 때 전체 테이블을 검색할 필요 없이 해당 파티션을 삭제하면 됩니다. 이는 삭제 성능을 크게 향상시킵니다.
  - **데이터 아카이빙**: 특정 파티션을 아카이브하거나 이동하는 작업이 더 용이합니다.

### 3. 데이터 삭제를 자동화하는 방안

**자동화된 데이터 삭제**

- 스케줄러 또는 이벤트 트리거 사용

  :

  - **MySQL 이벤트 스케줄러**: MySQL의 이벤트 스케줄러를 사용하여 정기적으로 오래된 데이터를 삭제하는 자동화된 작업을 설정할 수 있습니다.

**예시: MySQL 이벤트 스케줄러를 사용한 데이터 삭제**

```
sql코드 복사CREATE EVENT delete_old_records
ON SCHEDULE EVERY 1 MONTH
DO
    DELETE FROM call_records
    WHERE call_date < DATE_SUB(CURDATE(), INTERVAL 1 YEAR);
```

- 스크립트 및 크론 작업

  :

  - **크론 작업(Cron Job)**: 리눅스 환경에서는 크론 작업을 설정하여 정기적으로 데이터 삭제를 수행하는 스크립트를 실행할 수 있습니다.

**예시: 크론 작업을 사용한 데이터 삭제**

1. **삭제 스크립트 작성** (e.g., `delete_old_records.sql`):

```
sql코드 복사DELETE FROM call_records
WHERE call_date < DATE_SUB(CURDATE(), INTERVAL 1 YEAR);
```

1. **크론 작업 설정**:

```
bash코드 복사# 크론탭 파일을 열어 작업 추가
crontab -e

# 매월 1일 자정에 삭제 스크립트 실행
0 0 1 * * mysql -u username -p password database_name < /path/to/delete_old_records.sql
```

### 결론

1. **월별 조회 성능 향상 및 데이터 삭제**:
   - **파티셔닝**: 테이블을 월별로 파티셔닝하여 조회 성능을 개선하고, 데이터 삭제를 효율적으로 수행합니다.
2. **장점**:
   - **성능 향상**: 데이터 검색 및 인덱스 탐색 성능 향상.
   - **데이터 관리**: 데이터 삭제 및 아카이빙이 용이함.
3. **자동화된 데이터 삭제**:
   - **MySQL 이벤트 스케줄러** 또는 **크론 작업**을 사용하여 정기적으로 오래된 데이터를 자동으로 삭제합니다.

<br>

------



**시나리오 5**

**상황:** K 통신사는 자사의 모바일 앱을 사용하는 수백만 명의 고객에게 중요한 공지 사항과 프로모션 정보를 실시간으로 전달하기 위해 Push 알림 서비스를 제공하고 있습니다. K 통신사는 대규모 고객에게 효율적으로 Push 알림을 전송하면서도 서버 부하를 최소화하고, 메시지가 신속하게 전달되도록 해야 합니다. 알림을 제공하기 위해 웹소켓(WebSocket) 기술을 도입하려고 합니다.

**문제:**

1. 웹소켓을 사용하여 서버와 클라이언트 간 실시간 통신을 설정하는 기본적인 방법과 단계를 설명하세요.**
   ****힌트**: 서버는 WebSocket 서버를 시작하고, 클라이언트는 WebSocket 객체를 생성하여 서버에 연결합니다. 연결이 성공하면 서버와 클라이언트 간 실시간 메시지 전송이 가능합니다.
2. 웹소켓 연결에서 발생할 수 있는 보안 문제들을 제시하고, 각각의 문제 해결 방안을 제시하세요.
   **힌트**: XSS(교차 사이트 스크립팅) 공격을 방지하기 위해 데이터 검증 및 인코딩을 수행하고, WebSocket 연결을 SSL/TLS를 통해 암호화하여 데이터 전송 시 보안을 강화하세요.

<br>**문제 1: 웹소켓을 사용하여 서버와 클라이언트 간 실시간 통신을 설정하는 방법**

### 기본적인 방법과 단계

**1. 서버에서 WebSocket 서버 시작**

서버에서 WebSocket 서버를 설정하여 클라이언트의 연결을 수립할 준비를 합니다. WebSocket 서버는 클라이언트의 연결 요청을 수락하고, 실시간으로 메시지를 송수신할 수 있도록 합니다.

**예시 (Node.js + ws 라이브러리):**

```
javascript코드 복사const WebSocket = require('ws');
const server = new WebSocket.Server({ port: 8080 });

server.on('connection', ws => {
    console.log('Client connected');

    ws.on('message', message => {
        console.log('Received:', message);
        // Echo the message back to the client
        ws.send(`Server received: ${message}`);
    });

    ws.on('close', () => {
        console.log('Client disconnected');
    });
});
```

**2. 클라이언트에서 WebSocket 객체 생성 및 서버에 연결**

클라이언트는 `WebSocket` 객체를 생성하고, 서버 URL을 통해 연결을 시도합니다. 연결이 성공하면 메시지를 전송하거나 수신할 수 있습니다.

**예시 (JavaScript):**

```
javascript코드 복사const socket = new WebSocket('ws://localhost:8080');

socket.onopen = () => {
    console.log('Connected to server');
    socket.send('Hello Server!');
};

socket.onmessage = event => {
    console.log('Message from server:', event.data);
};

socket.onclose = () => {
    console.log('Disconnected from server');
};
```

**3. 실시간 메시지 전송**

서버와 클라이언트는 연결이 성립되면 실시간으로 메시지를 주고받을 수 있습니다. 서버는 `ws.send()` 메서드를 사용하여 메시지를 클라이언트에 전송하고, 클라이언트는 `socket.send()` 메서드를 사용하여 서버에 메시지를 보낼 수 있습니다.

### 문제 2: 웹소켓 연결에서 발생할 수 있는 보안 문제 및 해결 방안

**1. XSS (Cross-Site Scripting) 공격**

- **문제**: 악의적인 사용자가 클라이언트 측 코드에 스크립트를 삽입하여 서버와의 연결을 통해 악성 코드를 실행할 수 있습니다.

- 해결 방안

  :

  - **입력 검증**: 서버는 클라이언트에서 받은 모든 데이터를 검증하고, 기대되는 형식과 값만 허용합니다.
  - **출력 인코딩**: 클라이언트에서 데이터를 표시할 때, HTML 특수 문자를 이스케이프하여 스크립트가 실행되지 않도록 합니다.

**2. 데이터 전송 시 보안 문제**

- **문제**: WebSocket 연결이 암호화되지 않으면 중간자 공격(MITM)으로 인해 데이터가 탈취되거나 조작될 수 있습니다.

- 해결 방안

  :

  - **SSL/TLS 사용**: WebSocket Secure (wss://) 프로토콜을 사용하여 데이터를 암호화합니다. 이는 HTTPS와 유사한 방식으로, 데이터 전송 시 암호화를 제공합니다.

**예시: TLS를 사용한 WebSocket 연결**

서버에서 SSL/TLS를 설정한 후 `wss://` 프로토콜을 사용하여 연결합니다.

**서버 (Node.js + ws 라이브러리 + https):**

```
javascript코드 복사const fs = require('fs');
const https = require('https');
const WebSocket = require('ws');

// HTTPS 서버 설정
const server = https.createServer({
    key: fs.readFileSync('path/to/your/private.key'),
    cert: fs.readFileSync('path/to/your/certificate.crt')
});
const wss = new WebSocket.Server({ server });

wss.on('connection', ws => {
    console.log('Client connected');

    ws.on('message', message => {
        console.log('Received:', message);
        ws.send(`Server received: ${message}`);
    });

    ws.on('close', () => {
        console.log('Client disconnected');
    });
});

server.listen(8080, () => {
    console.log('Server is listening on port 8080');
});
```

**클라이언트:**

```
javascript코드 복사const socket = new WebSocket('wss://yourserverdomain.com:8080');

socket.onopen = () => {
    console.log('Connected to server');
    socket.send('Hello Server!');
};

socket.onmessage = event => {
    console.log('Message from server:', event.data);
};

socket.onclose = () => {
    console.log('Disconnected from server');
};
```

**3. 인증 및 권한 관리**

- **문제**: 인증되지 않은 사용자가 접근할 수 있는 경우 민감한 데이터에 접근할 수 있습니다.

- 해결 방안

  :

  - **토큰 기반 인증**: WebSocket 연결 시 토큰을 사용하여 인증합니다. 예를 들어, JWT(JSON Web Token)를 사용할 수 있습니다.
  - **세션 관리**: 사용자 세션을 관리하여 권한이 없는 사용자가 데이터에 접근하지 못하도록 합니다.

**예시: JWT를 사용한 인증**

**서버**:

```
javascript코드 복사const WebSocket = require('ws');
const jwt = require('jsonwebtoken');
const SECRET_KEY = 'your_secret_key';

const server = new WebSocket.Server({ port: 8080 });

server.on('connection', ws => {
    // WebSocket 연결 시 JWT 토큰 검사
    ws.on('message', message => {
        let token;
        try {
            const data = JSON.parse(message);
            token = data.token;
            jwt.verify(token, SECRET_KEY);
            ws.send('Token is valid');
        } catch (e) {
            ws.send('Invalid token');
            ws.close();
        }
    });
});
```

**클라이언트**:

```
javascript코드 복사const token = 'your_jwt_token';  // 서버에서 발급받은 JWT
const socket = new WebSocket('ws://localhost:8080');

socket.onopen = () => {
    socket.send(JSON.stringify({ token: token }));
};

socket.onmessage = event => {
    console.log('Message from server:', event.data);
};

socket.onclose = () => {
    console.log('Disconnected from server');
};
```

### 결론

1. **웹소켓을 통한 실시간 통신 설정**:
   - 서버에서 WebSocket 서버를 설정하고 클라이언트에서 WebSocket 객체를 생성하여 실시간으로 메시지를 송수신합니다.
2. **보안 문제 및 해결 방안**:
   - **XSS**: 입력 검증 및 출력 인코딩을 통해 방지합니다.
   - **데이터 전송 보안**: SSL/TLS를 사용하여 WebSocket 연결을 암호화합니다.
   - **인증**: JWT와 같은 토큰 기반 인증을 사용하여 보안을 강화합니다.

<br>

------



**시나리오 6**

**상황:** 통신사 K는 고객 서비스 웹 애플리케이션의 성능을 최적화하고자 합니다. 이 웹 애플리케이션은 다양한 데이터 조회 및 계산 작업을 수행하며, 많은 수의 고객 요청을 처리해야 합니다. 특히, 특정 고객 정보 조회 및 통계 계산이 빈번히 발생하며, 이로 인해 데이터베이스에 큰 부하가 가해지고 있습니다. 이를 해결하기 위해 메모리 캐시를 활용하여 성능을 최적화하고자 합니다.

**문제:**

1. 캐시의 효율성을 높이기 위해 고려해야 할 캐시 갱신 정책과 캐시 만료 정책을 설명하세요.
   **힌트**: LRU(Least Recently Used), LFU(Least Frequently Used)와 같은 캐시 갱신 정책을 고려하세요. 캐시 만료 정책으로는 TTL(Time-To-Live)이나 만료 시간 설정을 통해 데이터의 최신성을 유지할 수 있습니다.



------



**시나리오 7**

**상황: 통신사 K는 고객 정보를 관리하기 위해 RESTful API를 설계하고자 합니다. 이 API는 고객 생성(Create), 조회(Read), 수정(Update), 삭제(Delete)와 같은 기본 CRUD 기능을 제공해야 합니다. API는 클라이언트 애플리케이션과 통신하여 데이터를 주고받으며, 보안 및 성능 또한 고려해야 합니다.**

**문제:**

1. RESTful API를 설계할 때 고려해야 할 주요 요소를 설명하세요.
   **힌트**: 리소스(URI)설계, HTTP메서드(GET,POST,PUT), 상태코드, 데이터 형식, 보안 등을 고려해야 합니다.
2. API의 보안을 강화하기 위한 방법을 설명하고, 예를 들어 설명하세요.
   **힌트**: OAuth2, JWT를 사용한 토큰 기반 인증, HTTPS를 통한 데이터 암호화, 입력 검증 및 데이터 인코딩을 통해 API 보안을 강화할 수 있습니다.

<br>**문제 1: RESTful API를 설계할 때 고려해야 할 주요 요소**

RESTful API를 설계할 때는 다음과 같은 주요 요소를 고려해야 합니다:

### 1. **리소스 설계 (URI 설계)**

- **리소스**: RESTful API의 모든 데이터는 리소스로 간주됩니다. 각 리소스는 고유한 URI(Uniform Resource Identifier)로 식별됩니다.

- URI 구조

  : 간결하고 의미 있는 URI를 설계해야 합니다. 예를 들어, 고객 정보를 관리하는 API의 경우 URI는 다음과 같이 설계할 수 있습니다:

  - `GET /customers` - 모든 고객 목록 조회
  - `GET /customers/{customer_id}` - 특정 고객 조회
  - `POST /customers` - 새 고객 생성
  - `PUT /customers/{customer_id}` - 기존 고객 정보 수정
  - `DELETE /customers/{customer_id}` - 특정 고객 삭제

### 2. **HTTP 메서드**

- **GET**: 리소스를 조회할 때 사용합니다.
- **POST**: 새로운 리소스를 생성할 때 사용합니다.
- **PUT**: 기존 리소스를 업데이트할 때 사용합니다.
- **DELETE**: 리소스를 삭제할 때 사용합니다.
- 각 메서드는 리소스 상태를 나타내며, 이러한 상태 전이(State Transition)에 따라 URI에 명확히 매핑되어야 합니다.

### 3. **상태 코드 (HTTP Status Codes)**

- API의 응답에는 HTTP 상태 코드를 포함하여 클라이언트에게 요청의 결과를 알립니다.
  - **200 OK**: 요청이 성공적으로 처리됨.
  - **201 Created**: 리소스가 성공적으로 생성됨.
  - **400 Bad Request**: 잘못된 요청(예: 유효하지 않은 입력).
  - **401 Unauthorized**: 인증이 필요하거나 실패함.
  - **404 Not Found**: 요청한 리소스를 찾을 수 없음.
  - **500 Internal Server Error**: 서버에서 요청을 처리하는 동안 오류가 발생함.

### 4. **데이터 형식 (Content Negotiation)**

- 클라이언트와 서버 간의 데이터 형식은 일반적으로 JSON이나 XML을 사용합니다. JSON이 더 널리 사용되며 가독성이 좋습니다.
- API 응답은 `Content-Type` 헤더를 통해 데이터 형식을 지정해야 합니다. 예를 들어, `application/json`은 JSON 형식을 나타냅니다.

### 5. **보안**

- **HTTPS**: 모든 API 통신은 HTTPS를 통해 암호화하여 전송됩니다.
- **인증 및 권한 부여**: API 접근을 제어하기 위해 인증 및 권한 부여 메커니즘을 구현해야 합니다. OAuth2나 JWT 같은 토큰 기반 인증을 사용하여 보안을 강화합니다.

### 6. **버전 관리**

- API는 지속적으로 변경될 수 있으므로 버전 관리를 통해 호환성을 유지해야 합니다. URI에 버전을 포함하는 방식이 일반적입니다. 예를 들어, `GET /v1/customers`는 1버전의 API를 호출하는 것입니다.

**문제 2: API의 보안을 강화하기 위한 방법**

RESTful API의 보안을 강화하기 위해서는 다음과 같은 방법들을 사용할 수 있습니다:

### 1. **OAuth2를 사용한 인증 및 권한 부여**

- **OAuth2**는 토큰 기반의 인증 프로토콜로, 사용자나 애플리케이션이 API에 접근할 수 있는 권한을 안전하게 부여합니다.
- **액세스 토큰**: 인증된 클라이언트는 API 요청 시 액세스 토큰을 헤더(`Authorization: Bearer <token>`)에 포함시켜 보냅니다. 서버는 토큰을 검증하여 요청을 처리합니다.
- **리프레시 토큰**: 액세스 토큰이 만료된 경우, 리프레시 토큰을 사용하여 새로운 액세스 토큰을 발급받을 수 있습니다.

### 2. **JWT(JSON Web Token)를 사용한 토큰 기반 인증**

- **JWT**는 클라이언트가 서버에 로그인한 후, 서버에서 서명된 토큰을 발급하고 클라이언트는 이 토큰을 사용해 인증된 요청을 보낼 수 있습니다.

- **JWT 구조**: 헤더(Header), 페이로드(Payload), 서명(Signature)으로 구성됩니다. 서명은 토큰의 무결성을 검증하는 데 사용됩니다.

- 예를 들어, 로그인 후 클라이언트는 다음과 같은 형식으로 JWT를 사용하여 API에 요청을 보낼 수 있습니다:

  ```
  http코드 복사GET /api/protected_resource
  Authorization: Bearer <jwt_token>
  ```

### 3. **HTTPS를 통한 데이터 암호화**

- **HTTPS**는 클라이언트와 서버 간의 통신을 암호화하여 중간자 공격(MITM)이나 데이터 탈취를 방지합니다.
- 모든 API 요청과 응답은 HTTPS를 통해 이루어져야 하며, 클라이언트는 `https://`로 시작하는 URL을 사용하여 서버에 요청합니다.

### 4. **입력 검증 및 데이터 인코딩**

- **SQL 인젝션, XSS**와 같은 공격을 방지하기 위해 클라이언트에서 받은 모든 데이터를 철저히 검증하고 인코딩합니다.
- 예를 들어, SQL 인젝션을 방지하기 위해 Prepared Statements(준비된 문)를 사용하여 SQL 쿼리를 실행하고, XSS 공격을 방지하기 위해 출력 데이터를 HTML 엔티티로 인코딩합니다.

### 예시:

```
http코드 복사POST /customers
Authorization: Bearer <jwt_token>
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john.doe@example.com",
  "phone": "+123456789"
}
```

이 요청에서 JWT 토큰을 사용한 인증이 이루어지며, HTTPS를 통해 암호화된 채 전송됩니다. 서버는 JWT 토큰을 검증하여 사용자가 인증된 사용자임을 확인하고, JSON 데이터를 검증한 후 데이터베이스에 안전하게 저장합니다.

### 결론

1. **RESTful API 설계 요소**: 리소스 설계, HTTP 메서드 사용, 상태 코드, 데이터 형식, 보안, 버전 관리를 고려하여 설계해야 합니다.
2. **API 보안 강화 방법**: OAuth2와 JWT를 통한 토큰 기반 인증, HTTPS를 통한 데이터 암호화, 입력 검증 및 데이터 인코딩을 통해 API 보안을 강화합니다.

<br>

------



**시나리오 8**

**상황: 통신사 K는 사내 관리 시스템에서 역할 기반 접근 제어(권한관리)를 도입하려고 합니다. RBAC를 통해 직원들이 자신의 역할에 맞는 기능과 데이터에만 접근할 수 있도록 제한하려고 합니다. 현재 시스템에는 다양한 역할(예: 관리자, 매니저, 직원)이 있으며, 각 역할에 따라 접근 권한이 달라야 합니다.**

**문제:**

1. 역할 기반 접근 제어(RBAC)의 개념을 설명하고, RBAC의 주요 구성 요소(역할, 사용자, 권한)에 대해 설명하세요.
   **힌트**: RBAC는 사용자에게 역할을 부여하고, 역할에 권한을 부여하여 접근을 제어하는 시스템입니다. 역할(Role)은 특정 권한(Privilege)을 가지며, 사용자는 하나 이상의 역할을 가질 수 있습니다.

2. 통신사 K의 사내 관리 시스템에 RBAC를 적용하기 위한 데이터베이스(역할, 사용자, 권한) 테이블을 설계하고 ERD를 작성하세요.
   **힌트**: 사용자 테이블(Users), 역할 테이블(Roles), 권한 테이블(Permissions)과 사용자-역할(UserRoles), 역할-권한(RolePermissions) 관계 테이블을 설계하세요. ERD를 통해 각 테이블 간 관계를 시각화하세요.<br><br>**문제 1: 역할 기반 접근 제어(RBAC)의 개념 설명 및 주요 구성 요소 설명**

   ### 1. **역할 기반 접근 제어(RBAC) 개념**

   - **RBAC(Role-Based Access Control)**는 시스템의 보안을 강화하기 위해 사용자에게 부여된 역할(Role)에 따라 시스템 리소스에 대한 접근 권한을 제어하는 방법입니다. 사용자는 역할에 따라 권한을 할당받으며, 역할은 특정 작업이나 데이터에 대한 접근을 허용하는 권한(Privilege)을 포함합니다. 이를 통해 권한 관리가 용이해지고, 사용자마다 개별적으로 권한을 설정할 필요 없이 역할을 통해 일관되게 접근 권한을 관리할 수 있습니다.

   ### 2. **RBAC의 주요 구성 요소**

   - **역할(Role)**: 역할은 하나 이상의 권한을 묶어놓은 집합입니다. 예를 들어, "관리자" 역할은 사용자를 생성하고 삭제할 수 있는 권한을 포함할 수 있으며, "직원" 역할은 자신의 프로필만 조회할 수 있는 권한을 가질 수 있습니다.
   - **사용자(User)**: 사용자는 시스템을 사용하는 사람이나 프로그램을 의미합니다. 각 사용자는 하나 이상의 역할을 가질 수 있습니다. 예를 들어, 한 사용자는 "관리자"와 "매니저" 역할을 동시에 가질 수 있습니다.
   - **권한(Privilege/Permission)**: 권한은 특정 자원이나 기능에 대한 접근 권한을 정의합니다. 예를 들어, "사용자 관리" 권한은 사용자를 생성, 수정, 삭제할 수 있는 권한을 의미합니다.

   **문제 2: 통신사 K의 사내 관리 시스템에 RBAC를 적용하기 위한 데이터베이스 설계 및 ERD 작성**

   ### 1. **RBAC 데이터베이스 설계**

   RBAC를 적용하기 위해서는 사용자(User), 역할(Role), 권한(Permission) 테이블과 사용자-역할(UserRoles), 역할-권한(RolePermissions) 간의 관계를 나타내는 테이블이 필요합니다.

   - **Users 테이블**
     - **user_id**: 고유 식별자 (Primary Key)
     - **username**: 사용자 이름
     - **password**: 사용자 비밀번호(해시 형태로 저장)
     - **email**: 사용자 이메일
   - **Roles 테이블**
     - **role_id**: 고유 식별자 (Primary Key)
     - **role_name**: 역할 이름 (예: 관리자, 매니저, 직원)
   - **Permissions 테이블**
     - **permission_id**: 고유 식별자 (Primary Key)
     - **permission_name**: 권한 이름 (예: 사용자 관리, 데이터 조회)
   - **UserRoles 테이블**
     - **user_role_id**: 고유 식별자 (Primary Key)
     - **user_id**: Users 테이블의 외래 키 (Foreign Key)
     - **role_id**: Roles 테이블의 외래 키 (Foreign Key)
   - **RolePermissions 테이블**
     - **role_permission_id**: 고유 식별자 (Primary Key)
     - **role_id**: Roles 테이블의 외래 키 (Foreign Key)
     - **permission_id**: Permissions 테이블의 외래 키 (Foreign Key)

   ### 2. **ERD (Entity-Relationship Diagram) 설계**

   **ERD**는 각 테이블 간의 관계를 시각적으로 표현합니다:

   ```
   plaintext코드 복사Users               UserRoles              Roles              RolePermissions          Permissions
   +------------+      +------------+         +------------+      +----------------+       +------------+
   | user_id    |<---->| user_id    |         | role_id    |<---->| role_id        |       | permission_id |
   | username   |      | role_id    |<----+   | role_name  |      | permission_id  |<----+ | permission_name|
   | password   |      +------------+     |   +------------+      +----------------+     | +------------+
   | email      |                        |                                           |
   +------------+                        +-------------------------------------------+
   ```

   ### **ERD 설명**

   - **Users 테이블**: 사용자를 정의하며, 각 사용자는 고유한 `user_id`를 가집니다.
   - **Roles 테이블**: 역할을 정의하며, 각 역할은 고유한 `role_id`를 가집니다.
   - **Permissions 테이블**: 시스템 내의 권한을 정의하며, 각 권한은 고유한 `permission_id`를 가집니다.
   - **UserRoles 테이블**: 사용자와 역할 간의 다대다(Many-to-Many) 관계를 정의합니다. 사용자 하나가 여러 역할을 가질 수 있으며, 역할 하나를 여러 사용자가 가질 수 있습니다.
   - **RolePermissions 테이블**: 역할과 권한 간의 다대다(Many-to-Many) 관계를 정의합니다. 한 역할은 여러 권한을 가질 수 있으며, 하나의 권한이 여러 역할에 할당될 수 있습니다.

   ### **결론**

   RBAC를 통해 시스템의 보안을 강화하고, 사용자가 역할에 따라 적절한 권한만을 갖도록 할 수 있습니다. 데이터베이스 설계는 이러한 역할과 권한을 효과적으로 관리할 수 있도록 역할, 사용자, 권한, 그리고 이들 간의 관계를 테이블로 정의하고 ERD로 시각화하여 설계할 수 있습니다.

   <br>



------



**시나리오 9**

**상황:  K통신사의 가입, 해지, 기변 등의 오더 처리 화면을 대리점 및 상담사에게 제공하고 있습니다. 문제는 동일 고객의 정보를 서로다른 기기에서 업무처리를 하면서 계약 및 상품 정보의 데이터에 문제가 발생하고 있습니다.  **

![image-20240805224901628](/image-20240805224901628-1722865859905-2.png)

**문제:**

1. 서로다른 사용자가 동일 계약의 상품 정보를 동시에 변경하는 것을 방지하도록 방안을 제시하세요 

   

<br>**문제: 동일 계약의 상품 정보를 동시에 변경하는 것을 방지하기 위한 방안**

### 1. **문제 정의**

- **동시성 문제**: 동일한 고객의 계약 정보 또는 상품 정보를 여러 사용자가 동시에 업데이트하려고 할 때, 데이터 충돌이 발생하여 데이터 불일치나 손실이 생길 수 있습니다. 이는 특히 통신사의 계약이나 상품 관리와 같은 중요한 시스템에서 큰 문제가 될 수 있습니다.

### 2. **해결 방안 제시**

#### **a. 낙관적 잠금(Optimistic Locking)**

- **개념**: 데이터가 업데이트될 때까지 별도의 잠금을 걸지 않고, 업데이트 시점에 데이터가 다른 사용자에 의해 수정되었는지 확인합니다.
- **방법**:
  1. **버전 번호 필드 추가**: `contract` 테이블에 `version_number` 필드를 추가합니다. 이 필드는 데이터를 읽어올 때와 업데이트할 때 비교하여 동시 업데이트 여부를 확인하는 용도로 사용됩니다.
  2. **데이터 읽기**: 사용자가 계약 데이터를 조회할 때, 현재 `version_number`를 함께 읽어옵니다.
  3. **데이터 업데이트**: 데이터를 업데이트할 때 `UPDATE` 쿼리에 `WHERE version_number = ?` 조건을 추가하여 현재 버전이 일치하는 경우에만 업데이트가 진행되도록 합니다.
  4. **버전 불일치 시 처리**: 만약 `version_number`가 일치하지 않으면, 동시성 문제가 발생한 것이므로 사용자가 데이터를 다시 확인하고 수정할 수 있도록 안내합니다.
- **장점**: 동시 업데이트가 드물거나 사용자가 데이터를 자주 갱신하지 않는 경우에 효과적입니다.
- **단점**: 다중 사용자 환경에서 빈번한 갱신이 발생할 경우 충돌이 자주 발생하여 사용자 경험이 나빠질 수 있습니다.

#### **b. 비관적 잠금(Pessimistic Locking)**

- **개념**: 데이터에 대한 업데이트를 시작할 때, 다른 사용자가 해당 데이터를 수정하지 못하도록 잠금을 거는 방식입니다.
- **방법**:
  1. **잠금 설정**: 사용자가 특정 계약을 수정하기 위해 데이터를 읽어올 때 `SELECT ... FOR UPDATE` 쿼리를 사용하여 해당 데이터에 대해 잠금을 설정합니다.
  2. **데이터 수정**: 잠금이 설정된 상태에서 데이터 수정이 이루어지며, 수정이 완료되면 잠금이 해제됩니다.
  3. **동시 접근 방지**: 다른 사용자가 동일한 데이터를 수정하려고 할 때, 해당 데이터에 대한 잠금이 풀릴 때까지 대기하거나 수정이 불가능하다는 메시지를 표시합니다.
- **장점**: 데이터 무결성이 매우 중요한 환경에서 효과적입니다. 동시 업데이트가 많고 충돌 가능성이 높은 경우에 적합합니다.
- **단점**: 잠금이 오래 지속되면 시스템 성능 저하와 사용자 대기 시간이 증가할 수 있습니다.

#### **c. 기록 기반 잠금(Row-Level Locking)**

- **개념**: 특정 테이블의 특정 레코드에 대해 잠금을 설정하여, 해당 레코드에 대한 동시에 발생할 수 있는 갱신을 방지합니다.
- **방법**:
  1. **레코드 잠금**: 사용자가 계약 데이터를 수정할 때, 해당 레코드에 대해 레코드 기반 잠금을 설정합니다.
  2. **동시 갱신 차단**: 다른 사용자가 동일 레코드에 대해 수정 작업을 시도할 경우, 잠금이 풀릴 때까지 기다리거나 작업이 불가능하다는 알림을 받습니다.
  3. **업데이트 완료 후 잠금 해제**: 데이터 수정이 완료되면 잠금이 해제됩니다.
- **장점**: 비관적 잠금과 유사한 방식으로, 특정 레코드에 대한 충돌을 명확하게 방지할 수 있습니다.
- **단점**: 잠금이 오래 지속되면 성능에 영향을 미칠 수 있습니다.

### 3. **선택 기준 및 결론**

- **낙관적 잠금(Optimistic Locking)**은 동시 수정이 드문 경우에 적합하며, 성능 저하를 최소화할 수 있습니다. 사용자가 발생할 수 있는 충돌에 대해 인지하고 재시도를 하게 하는 방식입니다.
- **비관적 잠금(Pessimistic Locking)** 및 **기록 기반 잠금(Row-Level Locking)**은 데이터 충돌이 빈번하게 발생하거나, 데이터 무결성이 절대적으로 중요한 경우에 유리합니다. 이러한 방법은 데이터를 안전하게 보호하지만, 시스템 성능에 부담을 줄 수 있습니다.

따라서 K통신사의 사용 패턴과 데이터 무결성 요구 사항에 따라 적절한 방식을 선택하여 동시성 문제를 해결할 수 있습니다.<br>