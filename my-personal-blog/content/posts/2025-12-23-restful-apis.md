---
title: "RESTful APIs — Thiết kế API Web Hiện đại"
date: 2025-12-23T09:30:00+07:00
tags: ["rest", "api", "web-services"]
draft: false
summary: "Hướng dẫn thiết kế RESTful APIs - cách xây dựng web services hiệu quả."
---

## REST là gì?

REST (Representational State Transfer) là một kiến trúc software cho việc xây dựng web services. REST sử dụng HTTP methods để thực hiện các thao tác trên tài nguyên.

### RESTful Principles

1. **Client-Server Architecture**: Tách biệt giữa client và server
2. **Statelessness**: Mỗi request chứa đầy đủ thông tin
3. **Uniform Interface**: API có interface thống nhất
4. **Resource-oriented**: Tập trung vào resources, không phải actions
5. **Cacheable**: Response có thể cache được

## Resource vs Action

### Sai (RPC-style)

```
GET /api/getUser/123
GET /api/getUserList
POST /api/createUser
POST /api/deleteUser/123
POST /api/updateUser/123
```

**Vấn đề:** Lẫn lộn giữa action (verb) và resource (noun)

### Đúng (REST-style)

```
GET /api/users/123          # Get user 123
GET /api/users              # Get all users
POST /api/users             # Create new user
PUT /api/users/123          # Update user 123
DELETE /api/users/123       # Delete user 123
```

**Tốt hơn:** Rõ ràng về resource và method

## HTTP Methods trong REST

### GET - Lấy dữ liệu

```
GET /api/users/123
GET /api/users?page=1&limit=10
GET /api/users?role=admin
```

**Đặc điểm:**
- Idempotent (gọi nhiều lần kết quả giống nhau)
- Safe (không thay đổi dữ liệu)
- Cacheable

### POST - Tạo dữ liệu

```
POST /api/users
Content-Type: application/json

{
  "name": "John",
  "email": "john@example.com"
}
```

**Response:**
```
201 Created
Location: /api/users/123

{
  "id": 123,
  "name": "John",
  "email": "john@example.com"
}
```

**Đặc điểm:**
- Không idempotent (gọi 2 lần tạo 2 users)
- Không safe
- Không cacheable (trong hầu hết trường hợp)

### PUT - Cập nhật hoàn toàn

```
PUT /api/users/123
Content-Type: application/json

{
  "name": "Jane",
  "email": "jane@example.com"
}
```

**Đặc điểm:**
- Idempotent (cập nhật cùng 1 nội dung bao nhiêu lần cũng như nhau)
- Thay thế toàn bộ resource

### PATCH - Cập nhật một phần

```
PATCH /api/users/123
Content-Type: application/json

{
  "name": "Jane"
}
```

**Đặc điểm:**
- Cập nhật chỉ các field được gửi
- Không phải lúc nào cũng idempotent

### DELETE - Xóa dữ liệu

```
DELETE /api/users/123
```

**Response:**
```
204 No Content
```

**Đặc điểm:**
- Idempotent (xóa 2 lần vẫn được)
- Không safe

## API Design Patterns

### Versioning

```
GET /v1/users
GET /v2/users        # Khác version
```

Hoặc với header:

```
GET /api/users
Accept-Version: v1
```

### Filtering, Sorting, Pagination

```
GET /api/users?role=admin&status=active    # Filter
GET /api/users?sort=-created_at             # Sort (descending)
GET /api/users?page=2&limit=20              # Pagination
```

### Nested Resources

```
GET /api/users/123/posts
GET /api/users/123/posts/456
POST /api/users/123/posts
PUT /api/users/123/posts/456
DELETE /api/users/123/posts/456
```

### Relationships

```
GET /api/users/123?include=posts,comments
```

## HTTP Status Codes trong REST

### 2xx - Success

```
200 OK              - Request thành công
201 Created         - Tạo mới resource
204 No Content      - Thành công, không có body
```

### 3xx - Redirection

```
301 Moved Permanently   - Moved vĩnh viễn
302 Found              - Moved tạm thời
304 Not Modified       - Có thể dùng cache
```

### 4xx - Client Error

```
400 Bad Request         - Request sai format
401 Unauthorized        - Cần xác thực
403 Forbidden           - Không có quyền
404 Not Found           - Resource không tồn tại
409 Conflict            - Xung đột (duplicate key...)
422 Unprocessable       - Dữ liệu không hợp lệ
```

### 5xx - Server Error

```
500 Internal Server Error    - Lỗi server
503 Service Unavailable      - Server maintenance
```

## Error Handling

### Cách sai

```json
HTTP 200 OK
{
  "message": "User not found"
}
```

### Cách đúng

```json
HTTP 404 Not Found
Content-Type: application/json

{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User with ID 123 not found",
    "details": {
      "userId": 123
    }
  }
}
```

## Authentication & Authorization

### Basic Auth

```
GET /api/protected
Authorization: Basic base64(username:password)
```

### Bearer Token (JWT)

```
GET /api/protected
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### API Key

```
GET /api/protected?api_key=abc123xyz
```

Hoặc với header:

```
GET /api/protected
X-API-Key: abc123xyz
```

## Ví dụ REST API với Java + Spring Boot

```java
import org.springframework.web.bind.annotation.*;
import org.springframework.http.HttpStatus;
import java.util.List;

@RestController
@RequestMapping("/api/users")
public class UserController {
    
    // GET all users
    @GetMapping
    public List<User> getAllUsers(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int limit) {
        return userService.findAll(page, limit);
    }
    
    // GET user by ID
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
    
    // POST create user
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public User createUser(@RequestBody UserRequest request) {
        return userService.save(request);
    }
    
    // PUT update user
    @PutMapping("/{id}")
    public User updateUser(
        @PathVariable Long id,
        @RequestBody UserRequest request) {
        return userService.update(id, request);
    }
    
    // PATCH update user
    @PatchMapping("/{id}")
    public User patchUser(
        @PathVariable Long id,
        @RequestBody UserPatchRequest request) {
        return userService.patch(id, request);
    }
    
    // DELETE user
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteUser(@PathVariable Long id) {
        userService.delete(id);
    }
}
```

## HATEOAS (Hypermedia As The Engine)

Cấp cao nhất của REST maturity:

```json
{
  "id": 123,
  "name": "John",
  "email": "john@example.com",
  "_links": {
    "self": { "href": "/api/users/123" },
    "all": { "href": "/api/users" },
    "delete": { "href": "/api/users/123", "method": "DELETE" },
    "update": { "href": "/api/users/123", "method": "PUT" }
  }
}
```

## REST Maturity Model (Richardson)

```
Level 0: RPC (Tất cả là POST /api)
Level 1: Resources (Sử dụng URLs /api/users/123)
Level 2: HTTP Methods (GET, POST, PUT, DELETE)
Level 3: HATEOAS (Links đến resources khác)
```

## Best Practices

1. **Sử dụng HTTP methods chính xác**: GET (read), POST (create), PUT (update), DELETE (delete)
2. **Response thích hợp**: 201 Created, 204 No Content, 404 Not Found
3. **Versioning**: Hỗ trợ nhiều versions
4. **Documentation**: Sử dụng OpenAPI/Swagger
5. **Rate limiting**: Bảo vệ API khỏi abuse
6. **CORS**: Xử lý cross-origin requests
7. **Input validation**: Validate tất cả input
8. **Security**: HTTPS, authentication, authorization

## Kết luận

RESTful APIs là standard cho modern web development. Hiểu rõ REST principles sẽ giúp bạn:

✅ Thiết kế APIs cleaner và maintainable  
✅ Dễ dàng scale và extend  
✅ Khách hàng dễ dàng sử dụng API  
✅ Tích hợp với hệ thống khác dễ dàng  

REST không phức tạp, chỉ cần follow các nguyên tắc cơ bản!
