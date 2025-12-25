# HTTP Protocol và Web Communication — Từ Request đến Response


## HTTP Protocol là gì?

HTTP (HyperText Transfer Protocol) là giao thức cơ bản cho World Wide Web. Mỗi lần bạn truy cập một trang web, trình duyệt của bạn đang sử dụng HTTP để giao tiếp với server.

### Đặc điểm chính

- **Client-Server Model**: Client (trình duyệt) gửi request, Server trả lại response.
- **Stateless**: Mỗi request độc lập, server không lưu trữ thông tin về request trước.
- **Text-based**: HTTP sử dụng text ASCII, dễ đọc và debug.
- **Port mặc định**: Port 80 (HTTP), Port 443 (HTTPS).

## HTTP Request

### Cấu trúc Request

```
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0
Accept: text/html
```

**Gồm 4 phần:**

1. **Request Line**: `METHOD PATH HTTP/VERSION`
   - Method: GET, POST, PUT, DELETE, HEAD, OPTIONS...
   - Path: /index.html
   - Version: HTTP/1.1 hoặc HTTP/2

2. **Headers**: Các thông tin bổ sung
   - `Host`: Tên miền/máy chủ
   - `User-Agent`: Thông tin trình duyệt
   - `Content-Length`: Kích thước body
   - `Content-Type`: Loại dữ liệu (application/json, text/html...)

3. **Empty Line**: Dòng trống phân cách header và body

4. **Body**: Dữ liệu (chỉ có với POST, PUT)

### HTTP Methods

| Method | Mục đích | Body |
|--------|---------|------|
| **GET** | Lấy dữ liệu | Không |
| **POST** | Gửi dữ liệu | Có |
| **PUT** | Cập nhật dữ liệu | Có |
| **DELETE** | Xóa dữ liệu | Không |
| **HEAD** | Như GET nhưng không lấy body | Không |
| **OPTIONS** | Lấy các phương thức hỗ trợ | Không |

### Ví dụ GET Request

```
GET /api/users/123 HTTP/1.1
Host: api.example.com
Accept: application/json
```

### Ví dụ POST Request

```
POST /api/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Content-Length: 27

{"name":"Tùng","age":25}
```

## HTTP Response

### Cấu trúc Response

```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1234

<html>...</html>
```

**Gồm 4 phần:**

1. **Status Line**: `HTTP/VERSION STATUS_CODE STATUS_MESSAGE`
2. **Headers**: Thông tin phản hồi
3. **Empty Line**: Dòng trống
4. **Body**: Nội dung (HTML, JSON, ...)

### HTTP Status Codes

| Code | Ý nghĩa | Ví dụ |
|------|---------|-------|
| **1xx** | Informational | 100 Continue |
| **2xx** | Success | 200 OK, 201 Created |
| **3xx** | Redirection | 301 Moved Permanently, 302 Found |
| **4xx** | Client Error | 400 Bad Request, 404 Not Found, 403 Forbidden |
| **5xx** | Server Error | 500 Internal Error, 503 Service Unavailable |

### Các Status Code phổ biến

- **200 OK**: Request thành công
- **201 Created**: Tạo mới resource thành công
- **204 No Content**: Thành công nhưng không có nội dung trả lại
- **301 Moved Permanently**: Redirect vĩnh viễn
- **302 Found**: Redirect tạm thời
- **400 Bad Request**: Client gửi yêu cầu sai
- **401 Unauthorized**: Cần xác thực
- **403 Forbidden**: Không có quyền
- **404 Not Found**: Không tìm thấy resource
- **500 Internal Server Error**: Lỗi server

## HTTPS - HTTP Bảo mật

HTTPS = HTTP + TLS/SSL encryption

```
GET /secure-data HTTP/1.1
Host: secure.example.com
```

**Quy trình:**

1. Client gửi request HTTP thông thường
2. Server trả về certificate
3. Client xác minh certificate
4. Thiết lập SSL/TLS connection
5. Tất cả dữ liệu sau đó được mã hóa

## HTTP Headers quan trọng

### Request Headers

```
Host: example.com
User-Agent: Mozilla/5.0
Accept: application/json
Authorization: Bearer token123
Content-Type: application/json
Cookie: sessionid=abc123
```

### Response Headers

```
Content-Type: application/json
Content-Length: 1024
Set-Cookie: sessionid=abc123
Cache-Control: max-age=3600
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
Last-Modified: Mon, 23 Dec 2025 10:00:00 GMT
```

## Connection Management

### HTTP/1.1 Keep-Alive

Giữ kết nối mở để gửi nhiều requests:

```
GET /index.html HTTP/1.1
Host: example.com
Connection: keep-alive
```

### HTTP/2

- **Multiplexing**: Gửi nhiều requests trên 1 connection
- **Header Compression**: Giảm kích thước headers
- **Server Push**: Server gửi dữ liệu chủ động

## Ví dụ thực tế với Java

### Gửi HTTP Request

```java
import java.io.*;
import java.net.*;

public class HttpClient {
    public static void main(String[] args) throws Exception {
        URL url = new URL("http://api.example.com/data");
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();
        
        // Set request method
        conn.setRequestMethod("GET");
        conn.setRequestProperty("Accept", "application/json");
        
        // Get response code
        int statusCode = conn.getResponseCode();
        System.out.println("Status: " + statusCode);
        
        // Read response
        BufferedReader reader = new BufferedReader(
            new InputStreamReader(conn.getInputStream())
        );
        String line;
        while ((line = reader.readLine()) != null) {
            System.out.println(line);
        }
        reader.close();
    }
}
```

### Nhận HTTP Request (Simple Server)

```java
import java.io.*;
import java.net.*;

public class SimpleHttpServer {
    public static void main(String[] args) throws IOException {
        ServerSocket server = new ServerSocket(8080);
        System.out.println("Server running on port 8080");
        
        while (true) {
            Socket client = server.accept();
            BufferedReader reader = new BufferedReader(
                new InputStreamReader(client.getInputStream())
            );
            
            String requestLine = reader.readLine();
            System.out.println("Request: " + requestLine);
            
            // Send response
            PrintWriter writer = new PrintWriter(client.getOutputStream(), true);
            writer.println("HTTP/1.1 200 OK");
            writer.println("Content-Type: text/html");
            writer.println("Content-Length: 12");
            writer.println();
            writer.println("Hello World!");
            
            client.close();
        }
    }
}
```

## Kết luận

HTTP là giao thức cơ bản nhất cho web. Hiểu rõ HTTP request/response sẽ giúp bạn:

✅ Xây dựng web services  
✅ Debug vấn đề giao tiếp  
✅ Tối ưu hóa hiệu suất  
✅ Hiểu rõ RESTful APIs  

HTTP có vẻ đơn giản nhưng là nền tảng của toàn bộ web ngày nay!

