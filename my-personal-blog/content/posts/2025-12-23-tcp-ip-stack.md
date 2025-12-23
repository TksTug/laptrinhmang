---
title: "TCP/IP Stack — Hiểu Kiến trúc Internet"
date: 2025-12-23T10:30:00+07:00
tags: ["tcp-ip", "networking", "protocols"]
draft: false
summary: "Tìm hiểu kiến trúc TCP/IP 4 tầng - nền tảng của Internet."
---

## TCP/IP Model

TCP/IP là một mô hình phân tầng chỉ ra cách các giao thức mạng hoạt động với nhau.

### 4 Tầng của TCP/IP

```
┌─────────────────────────────────────────────────────┐
│ Layer 4: Application Layer                          │
│ (HTTP, HTTPS, FTP, SSH, DNS, SMTP, POP3...)        │
├─────────────────────────────────────────────────────┤
│ Layer 3: Transport Layer                            │
│ (TCP, UDP, SCTP)                                    │
├─────────────────────────────────────────────────────┤
│ Layer 2: Internet Layer                             │
│ (IP, ICMP, IGMPv4, IPv6)                            │
├─────────────────────────────────────────────────────┤
│ Layer 1: Link Layer                                 │
│ (Ethernet, PPP, ARP, WiFi drivers)                  │
└─────────────────────────────────────────────────────┘
```

## Layer 1: Link Layer (Data Link)

### Chức năng

- Truyền dữ liệu qua physical media (cáp, wifi)
- Xử lý MAC addresses
- Frame formatting

### Giao thức quan trọng

**Ethernet:**
```
┌─────┬──────┬──────┬─────────┬───────┐
│Preamble│Dest MAC│Src MAC│Type│ Data  │FCS│
│  8 B  │ 6 B  │ 6 B  │ 2 B │1500 B │4 B│
└─────┴──────┴──────┴─────────┴───────┘
```

**ARP (Address Resolution Protocol):**
```
ARP Request: "Who has IP 192.168.1.100?"
ARP Reply:   "I have it! MAC is 00:1A:2B:3C:4D:5E"
```

### MAC Address

```
Format: XX:XX:XX:XX:XX:XX (48 bits)
Ví dụ: 00:1A:2B:3C:4D:5E

Structure:
├─ First 3 octets: Manufacturer ID (OUI)
└─ Last 3 octets: Device ID
```

## Layer 2: Internet Layer

### Chức năng

- Routing dữ liệu qua mạng
- IP addressing
- Path finding

### IP Protocol

#### IPv4

```
┌──┬───┬────┬───┬──┬──┬────┬───────┬────┬─────────────┐
│Ver│IHL│DSCP│ECN│TTL│Flag│Offset │ID │Checksum │
│ 4 │ 4 │ 6  │ 2 │ 8 │ 3 │  13   │16 │   16    │
├──┼───┼────┼───┼──┼──┼────┼───────┼────┼─────────────┤
│       Source IP (32 bits)                           │
├────────────────────────────────────────────────────┤
│      Destination IP (32 bits)                      │
├────────────────────────────────────────────────────┤
│              Payload (Data)                         │
└────────────────────────────────────────────────────┘
```

**TTL (Time To Live):**
```
Ví dụ: TTL = 64
Mỗi khi đi qua một router, TTL -= 1
Khi TTL = 0, packet bị drop (tránh infinite loop)
```

#### IPv6

```
Format: 2001:0db8:85a3:0000:0000:8a2e:0370:7334
Rút gọn: 2001:db8:85a3::8a2e:370:7334
```

**Ưu điểm:**
- 128-bit address space (2^128 addresses - gần vô hạn)
- Simplified header
- Native IPSec support

### ICMP (Internet Control Message Protocol)

**Ping:**
```
ICMP Echo Request -> Server
Server -> ICMP Echo Reply
```

```bash
ping google.com
# ICMP Echo Request được gửi
# Server reply
# Kết quả: TTL=118, time=15ms
```

**Traceroute:**
```
Gửi packets với TTL tăng dần
TTL=1 -> Router 1 reply
TTL=2 -> Router 2 reply
TTL=3 -> Router 3 reply
...
TTL=n -> Destination reply
```

## Layer 3: Transport Layer

### Chức năng

- End-to-end communication
- Port-based addressing
- Flow control, reliability

### TCP (Transmission Control Protocol)

```
┌────┬────┬───────┬─────────┬────┬────┬──────┬────────┐
│Src │Dest│Seq No │Ack No   │Flags│Win │Check │Options │
│Port│Port│ (32)  │ (32)    │(9)  │Size│sum   │        │
└────┴────┴───────┴─────────┴────┴────┴──────┴────────┘
```

**Flags:**
- SYN: Synchronize sequence numbers
- ACK: Acknowledgment
- FIN: Finish (close connection)
- RST: Reset connection
- PSH: Push data immediately

**TCP Connection - 3-Way Handshake:**

```
Client                          Server
  |                               |
  |-------- SYN (seq=x) --------->|
  |                               |
  |<---- SYN-ACK (seq=y, ack=x+1)-|
  |                               |
  |------ ACK (seq=x+1, ack=y+1)->|
  |                               |
  |<---- Connection Established -->|
```

**Data Transfer:**

```
Client                          Server
  |                               |
  |---- Data (seq=x+1) ---------->|
  |                               |
  |<---- ACK (ack=x+1+len) --------|
  |                               |
```

**Connection Termination - 4-Way Handshake:**

```
Client                          Server
  |                               |
  |---------- FIN ------->|
  |                               |
  |<--------- ACK --------|
  |                               |
  |<--------- FIN --------|
  |                               |
  |---------- ACK ------->|
  |                               |
```

### UDP (User Datagram Protocol)

```
┌────┬────┬────────┬──────────┐
│Src │Dest│Length  │Checksum  │
│Port│Port│ (16)   │ (16)     │
└────┴────┴────────┴──────────┘
```

**Đặc điểm:**
- Connectionless (không cần establish connection)
- No reliability (best effort)
- Low overhead
- Fast

**TCP vs UDP:**

| Tiêu chí | TCP | UDP |
|----------|-----|-----|
| Connection | Required | Not required |
| Reliability | Guaranteed | Best effort |
| Order | Guaranteed | No |
| Speed | Slower | Faster |
| Use | HTTP, FTP, SSH | DNS, Video, Gaming |

## Layer 4: Application Layer

### Giao thức HTTP/HTTPS

```
Client                          Server
  |                               |
  |---- HTTP GET Request -------->|
  |                               |
  |<---- HTTP 200 Response --------|
  |                               |
```

### Giao thức SSH

Giống TCP nhưng có encryption:

```
Client                          Server
  |                               |
  |---- SSH Handshake ----------->|
  |                               |
  |<---- Server Key -------------|
  |                               |
  |---- Encrypted Auth ----------->|
  |                               |
  |<---- Session Established ------|
  |                               |
```

### Giao thức FTP

```
Control Connection (Port 21):
Client: USER username
Server: 331 Password required

Client: PASS password
Server: 230 Login successful

Client: LIST
Server: 150 Opening data connection
(Data Connection - Port 20)
Server: Data transferred...
Server: 226 Transfer complete
```

## Data Encapsulation

```
┌─────────────────────────────────────────┐
│        HTTP Request                     │  Layer 4
├─────────────────────────────────────────┤
│     TCP Header | HTTP Request           │  Layer 3
├─────────────────────────────────────────┤
│ IP Header | TCP Header | HTTP Request   │  Layer 2
├─────────────────────────────────────────┤
│ Ethernet Header | ... (as above) ...    │  Layer 1
└─────────────────────────────────────────┘
```

## Packet Journey

```
Application: "Send data"
     ↓
Transport (TCP): Add TCP header (ports, seq, ack)
     ↓
Internet (IP): Add IP header (source, dest IP)
     ↓
Link (Ethernet): Add Ethernet frame header (MAC)
     ↓
Physical: Send bits qua cáp/wifi
     ↓
[Network Media]
     ↓
Receiving end reverses process (De-encapsulation)
```

## Port Numbers

```
Well-known ports (0-1023):
- 21: FTP
- 22: SSH
- 25: SMTP
- 53: DNS
- 80: HTTP
- 110: POP3
- 143: IMAP
- 443: HTTPS

Registered ports (1024-49151):
- 3306: MySQL
- 5432: PostgreSQL
- 8080: HTTP Alternate

Dynamic ports (49152-65535):
- Assigned by OS for client connections
```

## Ví dụ Packet Dump

```
Ethernet II, Src: 00:1A:2B:3C:4D:5E, Dst: 00:11:22:33:44:55
Internet Protocol Version 4, Src: 192.168.1.100, Dst: 8.8.8.8
Transmission Control Protocol, Src Port: 54321, Dst Port: 443
Secure Sockets Layer
  Content Type: Application Data
  Version: TLS 1.2
  Length: 1234
  Encrypted data: [1234 bytes]
```

## Tools

```bash
# See all connections
netstat -an

# Detailed packet capture
tcpdump -i eth0 -w capture.pcap

# DNS lookup
nslookup google.com

# Traceroute
traceroute google.com

# Check ports
nmap -p 1-65535 localhost
```

## Kết luận

TCP/IP stack là nền tảng của Internet:

✅ Link Layer: Vật lý connection  
✅ Internet Layer: Routing & IP  
✅ Transport Layer: Reliability (TCP) vs Speed (UDP)  
✅ Application Layer: Services (HTTP, DNS, SSH...)  

Hiểu TCP/IP giúp bạn debug network issues, optimize performance, và thiết kế ứng dụng mạng tốt hơn!
