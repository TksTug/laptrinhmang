---
title: "DNS Protocol — Cách Internet Dịch Tên Miền thành IP"
date: 2025-12-23T09:00:00+07:00
tags: ["dns", "network-protocol", "internet"]
draft: false
summary: "Tìm hiểu cách DNS hoạt động, cấu trúc DNS queries, và cách phân giải tên miền."
---

## DNS là gì?

DNS (Domain Name System) là hệ thống phân giải tên miền trên Internet. Khi bạn gõ `google.com` vào trình duyệt, DNS sẽ dịch nó thành địa chỉ IP như `142.250.185.46`.

**Tại sao cần DNS?**

- Con người nhớ tên dễ hơn IP (google.com vs 142.250.185.46)
- IP có thể thay đổi, nhưng tên miền ổn định
- Dễ quản lý và scale

## Cấu trúc DNS

### Hierarchical Structure

```
                    . (Root)
                    |
        +---+---+---+---+---+
        |   |   |   |   |   |
       com  org  net  edu  vn
        |   |   |   |   |   |
      google yahoo  |   |   |
```

**Các cấp:**

1. **Root Domain** (.)
2. **Top-Level Domain** (TLD): .com, .org, .vn...
3. **Second-Level Domain**: google, facebook...
4. **Subdomain**: mail.google.com, api.github.com...

### DNS Record Types

| Record | Mục đích |
|--------|---------|
| **A** | IPv4 Address (192.168.1.1) |
| **AAAA** | IPv6 Address |
| **CNAME** | Canonical Name (Alias) |
| **MX** | Mail Exchange (Email Server) |
| **TXT** | Text Record (SPF, DKIM...) |
| **NS** | Name Server |
| **SOA** | Start of Authority |

### Ví dụ DNS Records

```
example.com.    300 IN A       192.168.1.1
www.example.com. 300 IN CNAME example.com.
example.com.    300 IN MX 10   mail.example.com.
example.com.    300 IN TXT    "v=spf1 mx ~all"
```

## DNS Query Process

### 1. Recursive Query

```
Client Browser
    |
    | 1. "What's IP of google.com?"
    v
Recursive Resolver (ISP)
    |
    | 2. "What's IP of google.com?"
    v
Root Nameserver
    |
    | 3. "Ask .com nameserver"
    v
TLD Nameserver (.com)
    |
    | 4. "Ask google's nameserver"
    v
Authoritative Nameserver
    |
    | 5. "IP is 142.250.185.46"
    v
Recursive Resolver
    |
    | 6. Returns IP to client
    v
Client Browser
```

**Bước chi tiết:**

1. **Local Query**: Trình duyệt kiểm tra cache cục bộ
2. **Recursive Resolver**: Nếu không có, hỏi ISP's resolver
3. **Root Nameserver**: Resolver hỏi root server
4. **TLD Nameserver**: Root chỉ đến TLD server
5. **Authoritative Nameserver**: TLD chỉ đến server có quyền
6. **Response**: Đáp án được trả lại qua mỗi bước

## DNS Packet Structure

### Query Packet

```
+-----+-----+-----+-----+-----+-----+-----+-----+
|                      ID                      |  (2 bytes)
+-----+-----+-----+-----+-----+-----+-----+-----+
|QR|  Opcode   |AA|TC|RD|RA| Z |RCODE         |  (2 bytes)
+-----+-----+-----+-----+-----+-----+-----+-----+
|                   QDCOUNT                    |  (2 bytes)
+-----+-----+-----+-----+-----+-----+-----+-----+
|                   ANCOUNT                    |  (2 bytes)
+-----+-----+-----+-----+-----+-----+-----+-----+
|                   NSCOUNT                    |  (2 bytes)
+-----+-----+-----+-----+-----+-----+-----+-----+
|                   ARCOUNT                    |  (2 bytes)
+-----+-----+-----+-----+-----+-----+-----+-----+
|                                             |
|              Question Section                |
|                                             |
+-----+-----+-----+-----+-----+-----+-----+-----+
```

## Caching và TTL

### TTL (Time To Live)

```
example.com.    3600 IN A 192.168.1.1
```

- TTL = 3600 giây (1 giờ)
- Cache lưu kết quả trong 1 giờ
- Sau 1 giờ, phải query lại

**Caching Levels:**

1. **Browser Cache**: 5-30 phút
2. **OS Cache**: 5-10 phút
3. **Resolver Cache**: 1-24 giờ
4. **Authoritative Server**: TTL được set

## DNS Security Issues

### DNS Spoofing

Attacker giả mạo response DNS để redirect traffic:

```
User queries: What's IP of bank.com?
Attacker responds: 192.168.1.100 (fake)
User connects to attacker's server
```

**Giải pháp:** DNSSEC (DNS Security Extensions)

### DNS Amplification Attack

Attacker sử dụng DNS server để tấn công DDoS:

```
Attacker -> DNS Server
("What's ALL DNS records for example.com?")
|
v
DNS Server -> Victim
(Large response - Amplification)
```

**Bảo vệ:** Rate limiting, query restrictions

## DNS Tools

### nslookup

```bash
nslookup google.com
nslookup -type=MX google.com
nslookup 142.250.185.46  # Reverse lookup
```

### dig (Domain Information Groper)

```bash
dig google.com
dig google.com +short
dig google.com MX
dig @8.8.8.8 google.com  # Query specific DNS server
```

### host

```bash
host google.com
host -t MX google.com
```

## Thực hành với Java

### DNS Query

```java
import java.net.*;

public class DNSLookup {
    public static void main(String[] args) throws UnknownHostException {
        String hostname = "google.com";
        
        // Forward lookup
        InetAddress address = InetAddress.getByName(hostname);
        System.out.println("Hostname: " + address.getHostName());
        System.out.println("IP: " + address.getHostAddress());
        
        // Get all addresses
        InetAddress[] addresses = InetAddress.getAllByName(hostname);
        System.out.println("\nAll IPs:");
        for (InetAddress addr : addresses) {
            System.out.println(addr.getHostAddress());
        }
        
        // Reverse lookup
        InetAddress ip = InetAddress.getByName("8.8.8.8");
        System.out.println("\nReverse lookup of 8.8.8.8:");
        System.out.println(ip.getHostName());
    }
}
```

### Custom DNS Client

```java
import java.net.*;
import java.io.*;

public class SimpleDNSClient {
    public static void main(String[] args) throws Exception {
        String hostname = "example.com";
        int port = 53;
        
        DatagramSocket socket = new DatagramSocket();
        
        // Tạo DNS query packet (simplified)
        byte[] query = createDNSQuery(hostname);
        
        DatagramPacket packet = new DatagramPacket(
            query, query.length,
            InetAddress.getByName("8.8.8.8"), port
        );
        
        socket.send(packet);
        
        // Receive response
        byte[] response = new byte[512];
        DatagramPacket responsePacket = new DatagramPacket(
            response, response.length
        );
        socket.receive(responsePacket);
        
        // Parse response (simplified)
        System.out.println("Got DNS response");
        
        socket.close();
    }
    
    private static byte[] createDNSQuery(String hostname) {
        // Simplified DNS query creation
        // In reality, need proper DNS packet format
        return new byte[0];
    }
}
```

## DNS Best Practices

1. **Sử dụng multiple DNS servers**: 8.8.8.8 (Google), 1.1.1.1 (Cloudflare)
2. **Implement DNS caching**: Giảm latency
3. **Sử dụng DNSSEC**: Bảo vệ khỏi spoofing
4. **Monitor DNS queries**: Phát hiện dị thường
5. **Thiết lập TTL phù hợp**: Balance giữa freshness và performance

## Kết luận

DNS là "phonebook của Internet". Mặc dù hoạt động phía sau, nó là yếu tố critical cho:

✅ Khám phá services  
✅ Load balancing  
✅ Failover  
✅ Bảo mật  

Hiểu DNS giúp bạn diagnose vấn đề mạng và tối ưu hóa ứng dụng web!
