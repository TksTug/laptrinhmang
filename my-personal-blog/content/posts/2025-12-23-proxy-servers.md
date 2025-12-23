---
title: "Proxy Servers & Network Intermediaries — Trung gian Mạng"
date: 2025-12-23T13:00:00+07:00
tags: ["proxy", "network", "intermediary"]
draft: false
summary: "Tìm hiểu về proxy servers, cách hoạt động, và ứng dụng thực tế trong network."
---

## Proxy Là Gì?

Proxy là một server đứng giữa client và server thật để làm trung gian:

```
┌────────┐        ┌───────────┐        ┌──────────┐
│ Client │ ────→ │ Proxy     │ ────→ │ Real     │
│        │ ←──── │ Server    │ ←──── │ Server   │
└────────┘        └───────────┘        └──────────┘
         (Thay đổi request/response)
```

## Types of Proxies

### 1. Forward Proxy

```
Client trên LAN muốn lên Internet:

Client
  │ Request www.google.com
  ▼
Forward Proxy (trên LAN)
  │ Real IP: 192.168.1.1, Proxy IP: 203.0.113.10
  ▼
Internet
  │ Google thấy request từ 203.0.113.10 (proxy)
  ▼
Google
```

**Use cases:**
- Cấm nhân viên truy cập websites nhất định
- Cache content cục bộ
- Ẩn IP address
- Scan malware

**Ví dụ config:**

```
Operating System Settings:
├─ Manual Proxy Configuration
│  └─ Proxy: 192.168.1.254
│     Port: 8080
│
Or environment variable:
export http_proxy=http://192.168.1.254:8080
```

### 2. Reverse Proxy

```
Internet
  │ Client request www.example.com
  ▼
Reverse Proxy (203.0.113.1)
  │ Load balance, cache, filter
  ├─ Server 1 (192.168.1.10:8080)
  ├─ Server 2 (192.168.1.11:8080)
  └─ Server 3 (192.168.1.12:8080)
```

**Use cases:**
- Load balancing
- Caching
- SSL termination
- Web acceleration
- Hide real servers

**Nginx Reverse Proxy:**

```nginx
server {
    listen 443 ssl;
    server_name example.com;
    
    ssl_certificate /etc/ssl/certs/cert.pem;
    ssl_certificate_key /etc/ssl/private/key.pem;
    
    # Reverse proxy configuration
    location / {
        proxy_pass http://backend_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

upstream backend_cluster {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}
```

### 3. Transparent Proxy

Client không biết có proxy:

```
┌────────┐
│ Client │  (Không cấu hình proxy)
└───┬────┘
    │ 
    ▼
┌────────────────────────┐
│ Network (Router)       │
│ ├─ Intercept traffic   │
│ ├─ Forward to proxy    │
│ └─ Transparent         │
└───┬────────────────────┘
    │
    ▼
┌───────────────────┐
│ Real Server       │
│ Không biết có     │
│ proxy intercepting│
└───────────────────┘
```

**Use case:** ISP monitoring, parental control

### 4. SOCKS Proxy

Hoạt động ở Layer 4 (Transport):

```
Protocol support:
├─ TCP traffic
└─ UDP traffic

Client
  │ CONNECT 203.0.113.1:443
  ▼
SOCKS Proxy
  │ (Tunnel any TCP/UDP connection)
  ▼
Target Server
```

**SSH SOCKS Tunnel:**

```bash
# Create SOCKS proxy over SSH
ssh -D 1080 user@remote-server.com

# Now traffic through localhost:1080 goes through SSH tunnel
# Configure browser to use SOCKS5 proxy: localhost:1080
```

## SSL/TLS Termination at Proxy

```
Without Proxy:
Client ──HTTPS──→ Server
         (Encryption overhead on server)

With Proxy:
Client ──HTTPS──→ Proxy ──HTTP──→ Server
       (Encrypted) (Load balanced, unencrypted inside LAN)

Benefits:
├─ Server doesn't handle encryption overhead
├─ Proxy inspects HTTPS traffic
└─ Better performance
```

**Configuration:**

```java
// Reverse proxy (Nginx) terminates SSL
location / {
    proxy_pass http://backend_server:8080;
    proxy_set_header X-Forwarded-Proto https;
    // Backend knows it's HTTPS from header
}

// Backend server
@RestController
public class ConfigController {
    @GetMapping("/api")
    public String api(@RequestHeader("X-Forwarded-Proto") String proto) {
        if (!"https".equals(proto)) {
            throw new SecurityException("Must use HTTPS!");
        }
        return "Secure response";
    }
}
```

## Proxy Caching

```
Request 1: GET /images/logo.png
  ├─ Not in cache
  ├─ Fetch from server
  └─ Store in cache (TTL: 24 hours)

Request 2-1000: GET /images/logo.png
  ├─ Found in cache
  └─ Serve immediately (fast!)
```

**HTTP Cache Headers:**

```
Server Response:
HTTP/1.1 200 OK
Cache-Control: public, max-age=86400
ETag: "123abc"
Last-Modified: Wed, 23 Dec 2025 10:00:00 GMT

Proxy/Browser:
├─ Cache for 24 hours
├─ After 24 hours, ask server
└─ If ETag matches, use cached version
```

**Java Implementation:**

```java
@RestController
public class ImageController {
    @GetMapping("/images/{filename}")
    public ResponseEntity<byte[]> getImage(@PathVariable String filename) {
        byte[] imageBytes = loadImage(filename);
        
        return ResponseEntity.ok()
            .cacheControl(CacheControl.maxAge(24, TimeUnit.HOURS).cachePublic())
            .eTag("\"" + computeHash(imageBytes) + "\"")
            .body(imageBytes);
    }
    
    @GetMapping("/images/{filename}")
    public ResponseEntity<Void> getImageIfModified(
        @PathVariable String filename,
        @RequestHeader(value = "If-None-Match", required = false) String etag
    ) {
        String currentETag = "\"" + computeHash(loadImage(filename)) + "\"";
        
        if (currentETag.equals(etag)) {
            return ResponseEntity.status(304).build(); // Not Modified
        }
        
        return ResponseEntity.ok().build();
    }
}
```

## Content Filtering & Blocking

```
Request: GET /adult-content
         │
         ▼
Proxy checks against blacklist
├─ Domain: adult-site.com (BLOCKED)
├─ Keywords in URL (DETECTED)
└─ Category: Adult (FORBIDDEN)
         │
         ▼
Proxy returns: 403 Forbidden

Client sees:
"This website is blocked by your administrator"
```

## HTTP/2 Server Push

```
Without proxy:
1. GET /index.html → Server
2. Browser parses HTML
3. GET /style.css → Server
4. GET /script.js → Server

With proxy (HTTP/2 push):
1. GET /index.html → Server
2. Server pushes /style.css, /script.js before asked
3. Browser receives all at once
```

**Config:**

```nginx
location /index.html {
    http2_push /style.css;
    http2_push /script.js;
    http2_push /image.png;
}
```

## Proxy with Authentication

```
Client → Proxy (authentication required)

Request:
GET /path HTTP/1.1
Proxy-Authorization: Basic dXNlcjpwYXNz...

Proxy:
├─ Decode credentials
├─ Verify against LDAP/AD
└─ If valid → Forward request
└─ If invalid → 407 Proxy Authentication Required
```

**Java Proxy Client:**

```java
import java.net.Proxy;
import java.net.ProxySelector;
import java.net.URI;
import java.net.URLConnection;

public class ProxyClientExample {
    public static void main(String[] args) throws Exception {
        // With authentication
        String proxyUrl = "http://user:password@proxy.example.com:8080";
        URI uri = URI.create(proxyUrl);
        
        Proxy proxy = new Proxy(
            Proxy.Type.HTTP,
            new InetSocketAddress(uri.getHost(), uri.getPort())
        );
        
        URLConnection conn = new URL("http://www.example.com").openConnection(proxy);
        
        // Add basic auth header
        String credentials = "user:password";
        String encoded = Base64.getEncoder().encodeToString(credentials.getBytes());
        conn.setRequestProperty("Proxy-Authorization", "Basic " + encoded);
        
        InputStream is = conn.getInputStream();
        // Read response
    }
}
```

## API Gateway (Advanced Proxy)

API Gateway = Proxy + Router + Authentication + Rate Limiting:

```
┌──────────────────────────────────────────────┐
│        API Gateway (Kong, AWS API Gateway)   │
├──────────────────────────────────────────────┤
│ ├─ Authentication (JWT, OAuth2)              │
│ ├─ Rate limiting (1000 req/min)              │
│ ├─ Request/Response transformation           │
│ ├─ Logging & Analytics                       │
│ ├─ CORS handling                             │
│ └─ Path-based routing                        │
└──────────┬──────────────────────────────────┘
           │
    ┌──────┼──────┬──────┐
    │      │      │      │
    ▼      ▼      ▼      ▼
 Auth   User  Product  Payment
 Service Service Service Service
```

**Example routing:**

```
GET /api/users/* → User Service
GET /api/products/* → Product Service
POST /api/auth/* → Auth Service
DELETE /api/payments/* → Payment Service (need admin role)
```

## Squid Proxy (Example)

Popular forward proxy:

```
# /etc/squid/squid.conf
http_port 3128

# ACL - Allow only office IPs
acl office_network src 192.168.1.0/24

# Block adult sites
acl blocked_sites dstdomain "/etc/squid/blocklist.txt"

http_access deny blocked_sites
http_access allow office_network
http_access deny all
```

## Performance Comparison

```
Direct Connection vs With Proxy:

Scenario: Fetch 100 MB file from internet

Direct:
├─ Download: 100 MB from internet (100 seconds)
├─ First user: 100 seconds
├─ Second user: 100 seconds
├─ Total: 200 seconds

With Proxy (caching):
├─ First user: 100 seconds (cached in proxy)
├─ Second user: 1 second (from proxy cache)
├─ Total: 101 seconds (99% faster!)
```

## Security Considerations

```
Proxy Security Risks:

1. Man-in-Middle (if proxy is compromised)
   └─ Attacker sees all traffic

2. Privacy concerns
   └─ Proxy admin can see where you go

3. Slow performance
   └─ All traffic goes through proxy

4. Single point of failure
   └─ If proxy down → No internet

Mitigation:
├─ Use HTTPS (encryption end-to-end)
├─ Trust only official proxies
├─ Use VPN instead for privacy
└─ Have failover proxy
```

## Kết luận

Proxy servers là công cụ quan trọng:

✅ Load balancing  
✅ Caching (faster)  
✅ Security filtering  
✅ Content inspection  
✅ Access control  

**Common usage:**
- Corporate networks: Forward proxy
- Web services: Reverse proxy
- Privacy: VPN proxy
- Performance: CDN (specialized proxy)

Hiểu rõ proxy giúp optimize network architecture và improve security!
