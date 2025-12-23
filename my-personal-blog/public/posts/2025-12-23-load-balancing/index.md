# Load Balancing & Scalability — Mở rộng Ứng dụng


## Load Balancing Là Gì?

Khi traffic tăng, một server không đủ. Load balancer phân phối requests đến nhiều servers:

```
                    Requests
                       │
                       ▼
                ┌───────────────┐
                │ Load Balancer │
                └───────┬───────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ▼               ▼               ▼
    ┌───────────┐ ┌───────────┐ ┌───────────┐
    │ Server 1  │ │ Server 2  │ │ Server 3  │
    │ (50% CPU) │ │ (45% CPU) │ │ (40% CPU) │
    └───────────┘ └───────────┘ └───────────┘
        │               │               │
        └───────────────┼───────────────┘
                        │
                        ▼
                   ┌─────────┐
                   │Database │
                   └─────────┘
```

## Load Balancing Algorithms

### 1. Round Robin

```
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1
Request 5 → Server 2
Request 6 → Server 3
...
```

**Ưu điểm:** Đơn giản  
**Nhược điểm:** Không xem xét tải thực tế của server

### 2. Least Connections

```
Server 1: 50 active connections
Server 2: 30 active connections  ← Next request goes here
Server 3: 45 active connections

Load Balancer picks server with least connections
```

**Ưu điểm:** Cân bằng tốt hơn  
**Nhược điểm:** Một request có thể tốn nhiều tài nguyên

### 3. Weighted Load Balancing

```
Server 1 (Powerful): 50% of traffic
Server 2 (Medium):   30% of traffic
Server 3 (Weak):     20% of traffic

Ratio: 5:3:2
```

**Use case:** Servers có sức mạnh khác nhau

### 4. IP Hash

```
User IP: 192.168.1.10
Hash(192.168.1.10) % 3 = 1
→ Always goes to Server 1 (for session persistence)
```

**Ưu điểm:** Session không mất, cache-friendly  
**Nhược điểm:** Nếu một server down, users phải reconnect

### 5. Least Response Time

```
Server 1: Avg response 50ms
Server 2: Avg response 100ms ← Previous request slower
Server 3: Avg response 80ms

Load Balancer picks Server 1 (fastest)
```

## Horizontal vs Vertical Scaling

### Vertical Scaling (Cộng Chiều)

```
1 Server with 2 CPU, 4 GB RAM
                ↓
Upgrade to 4 CPU, 16 GB RAM
```

**Ưu điểm:** Đơn giản  
**Nhược điểm:**
- Giới hạn vật lý (không thể upgrade vô hạn)
- Downtime khi nâng cấp
- Đắt tiền

### Horizontal Scaling (Mở Rộng Chiều)

```
1 Server with 2 CPU, 4 GB RAM
                ↓
2 Servers with 2 CPU, 4 GB RAM each
                ↓
3 Servers with 2 CPU, 4 GB RAM each
```

**Ưu điểm:**
- Không giới hạn (thêm servers khi cần)
- Không downtime
- Chi phí theo nhu cầu

**Nhược điểm:**
- Phức tạp (phải quản lý nhiều servers)
- Cần load balancer
- Data consistency khó hơn

## Popular Load Balancers

### 1. Nginx

```nginx
upstream backend {
    server backend1.example.com weight=5;
    server backend2.example.com:8080;
    server backend3.example.com backup;
}

server {
    listen 80;
    
    location / {
        proxy_pass http://backend;
    }
}
```

**Ưu điểm:**
- Open source
- Lightweight
- High performance
- Reverse proxy

### 2. HAProxy

```
# HAProxy config
listen web_cluster
    bind *:80
    balance roundrobin
    server web1 192.168.1.10:8080 check
    server web2 192.168.1.11:8080 check
    server web3 192.168.1.12:8080 check
```

**Ưu điểm:**
- Powerful configuration
- Good for L4-L7 balancing
- Active/passive health checks

### 3. AWS Load Balancer

```
Internet
  ↓
ALB (Application Load Balancer) - Layer 7
├─ Path-based routing
├─ Host-based routing
└─ Port-based routing
  ↓
┌─────────────────────────────┐
│    Auto Scaling Group       │
│ ┌──────┐ ┌──────┐ ┌──────┐ │
│ │ EC2  │ │ EC2  │ │ EC2  │ │
│ └──────┘ └──────┘ └──────┘ │
│ (Auto scales based on CPU)  │
└─────────────────────────────┘
```

## Health Checks

```
Load Balancer checks every 5 seconds:
Server 1: GET /health → 200 OK ✓ (Healthy)
Server 2: GET /health → 503 Timeout ✗ (Unhealthy)
Server 3: GET /health → 200 OK ✓ (Healthy)

New requests:
→ Server 1 and 3 only
→ Server 2 removed from pool until it recovers
```

**Health Check Endpoint:**

```java
@RestController
public class HealthController {
    @Autowired
    private DatabaseService db;
    
    @GetMapping("/health")
    public ResponseEntity<Health> health() {
        try {
            // Check database connection
            db.ping();
            return ResponseEntity.ok(
                new Health("UP", "Database OK")
            );
        } catch (Exception e) {
            return ResponseEntity.status(503).body(
                new Health("DOWN", "Database Error: " + e.getMessage())
            );
        }
    }
}
```

## Session Management

### Problem: Session Loss

```
Request 1 (User login)
→ Server 1: Creates session, stores session_id=abc123
→ User gets cookie: session_id=abc123

Request 2
→ Load Balancer routes to Server 2
→ Server 2: Doesn't have session abc123!
→ User must login again ❌
```

### Solution 1: Sticky Sessions

```
Load Balancer: 
"If session_id=abc123, always go to Server 1"

Cookie: session_id=abc123
└─ Always routes to Server 1
```

**Ưu điểm:** Đơn giản  
**Nhược điểm:**
- If Server 1 dies → User loses session
- Uneven load (some servers busier)

### Solution 2: Distributed Session Store

```
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Server 1 │  │ Server 2 │  │ Server 3 │
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │             │             │
     └─────────────┼─────────────┘
                   │
                   ▼
            ┌──────────────┐
            │ Redis Cluster│
            │(Session Store)
            └──────────────┘
```

**Code Example:**

```java
// Spring Session with Redis
@Configuration
@EnableRedisHttpSession
public class SessionConfig {
    // Spring auto-saves sessions to Redis
}

@RestController
public class LoginController {
    @PostMapping("/login")
    public String login(HttpSession session, String username) {
        session.setAttribute("username", username);
        // Saved to Redis automatically
        return "Login OK";
    }
    
    @GetMapping("/profile")
    public String profile(HttpSession session) {
        // Works on any server - data from Redis
        String username = (String) session.getAttribute("username");
        return "Welcome " + username;
    }
}
```

## Auto Scaling

```
                 Time
    ↑ CPU/Memory/Requests
    │
30% │                           ┌─────────
    │                           │
20% │    ┌────────────────┐     │
    │    │                │     │
10% │────┘                └─────┴──────
    └──────────────────────────────────→

During traffic spike:
├─ CPU jumps to 30%
├─ Load Balancer triggers: "Add 2 more servers"
├─ New servers added to pool
└─ CPU drops back to 20%
```

**AWS Auto Scaling Config:**

```
Target Group: web-servers
├─ Min servers: 2
├─ Max servers: 10
├─ Desired: 3
└─ Scaling Policy:
    ├─ Add server if CPU > 70% for 2 minutes
    └─ Remove server if CPU < 20% for 5 minutes
```

## Database Scaling

```
Single Database:
┌──────────────────────────────┐
│    Web Servers (3)           │
└──────────┬───────────────────┘
           │ All write to same DB
           ▼
     ┌───────────────┐
     │   Database    │ ← Bottleneck!
     │ (Single point)│
     └───────────────┘
```

### Solution 1: Read Replicas

```
Write requests
    │
    ▼
┌─────────────┐
│Primary DB   │──┐
└─────────────┘  │ Replicate
                 │
            ┌────┴────────┬────────────┐
            ▼             ▼            ▼
      ┌────────────┐ ┌──────────┐ ┌──────────┐
      │ Replica 1  │ │Replica 2 │ │Replica 3 │
      │(Read-only) │ │ (Read)   │ │ (Read)   │
      └────────────┘ └──────────┘ └──────────┘

Application:
├─ Write queries → Primary DB
└─ Read queries → Any replica
```

**Connection Pool:**

```java
// Write operations
HikariDataSource writeDS = new HikariDataSource();
writeDS.setJdbcUrl("jdbc:mysql://primary-db:3306/mydb");

// Read operations
HikariDataSource readDS = new HikariDataSource();
readDS.setJdbcUrl("jdbc:mysql://replica-1:3306/mydb");

// Usage
try (Connection write = writeDS.getConnection()) {
    // Write to primary
    write.createStatement().execute("INSERT INTO users VALUES...");
}

try (Connection read = readDS.getConnection()) {
    // Read from replica
    ResultSet rs = read.createStatement().executeQuery("SELECT * FROM users");
}
```

### Solution 2: Database Sharding

```
All users
│
├─ User ID 1-1000000
│  └─ Shard 1 (Database 1)
│
├─ User ID 1000001-2000000
│  └─ Shard 2 (Database 2)
│
└─ User ID 2000001-3000000
   └─ Shard 3 (Database 3)

Query for User 5000:
└─ Hash(5000) % 3 = 1 → Shard 1
```

**Implementation:**

```java
public class ShardingService {
    private static final int NUM_SHARDS = 3;
    private List<DataSource> shards;
    
    public DataSource getShardForUser(int userId) {
        int shardId = userId % NUM_SHARDS;
        return shards.get(shardId);
    }
    
    public User getUserById(int userId) {
        DataSource ds = getShardForUser(userId);
        try (Connection conn = ds.getConnection()) {
            String sql = "SELECT * FROM users WHERE id = ?";
            PreparedStatement stmt = conn.prepareStatement(sql);
            stmt.setInt(1, userId);
            ResultSet rs = stmt.executeQuery();
            // Parse result
        }
    }
}
```

## Caching Layer

```
Without Cache:
Request → Server → Database → 500ms response

With Cache:
Request → Cache (Redis) → 5ms response
If miss → Database → Update cache → 500ms

Request for popular product:
1st request: Cache miss → 500ms
2-1000th request: Cache hit → 5ms
```

**Implementation:**

```java
@RestController
@RequestMapping("/products")
public class ProductController {
    @Autowired
    private ProductService service;
    @Autowired
    private RedisTemplate<String, Product> redis;
    
    @GetMapping("/{id}")
    public Product getProduct(@PathVariable int id) {
        // Check cache first
        String key = "product:" + id;
        Product cached = redis.opsForValue().get(key);
        if (cached != null) {
            return cached; // 5ms
        }
        
        // Cache miss - fetch from DB
        Product product = service.findById(id);
        
        // Store in cache for 1 hour
        redis.opsForValue().set(key, product, Duration.ofHours(1));
        
        return product; // 500ms on first request
    }
}
```

## Monitoring & Observability

```
┌──────────────┐
│Application  │──┐
└──────────────┘  │
                  │ Metrics
┌──────────────┐  │
│Load Balancer │──┤
└──────────────┘  │
                  ▼
            ┌─────────────┐
            │ Prometheus  │ (Metrics collector)
            └──────┬──────┘
                   │
                ┌──┴──┐
                ▼     ▼
            Grafana Alerting
            (Dashboards) (PagerDuty)
```

**Metrics to Monitor:**

```
├─ Response Time (p50, p95, p99)
├─ Error Rate
├─ CPU/Memory usage per server
├─ Active connections
├─ Cache hit rate
├─ Database connections
└─ Network bandwidth
```

## Kết luận

Load balancing là essential cho production systems:

✅ Handle millions of requests  
✅ Automatic failover  
✅ Zero downtime deployments  
✅ Easy to scale  

**Typical Architecture:**

```
Users
  ↓
CDN (Cache static assets)
  ↓
Load Balancer (Nginx)
  ↓
App Servers (Horizontal scaling)
  ↓
Session Store (Redis)
  ↓
Cache Layer (Redis)
  ↓
Database (Primary + Replicas)
  ↓
Monitoring (Prometheus + Grafana)
```

Hệ thống scalable không phải xây một lần, mà là iterative process - start simple, monitor, scale khi cần!

