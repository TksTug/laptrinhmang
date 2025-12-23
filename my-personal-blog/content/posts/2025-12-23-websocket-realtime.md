---
title: "WebSocket — Real-time Communication với JavaScript"
date: 2025-12-23T10:00:00+07:00
tags: ["websocket", "javascript", "real-time"]
draft: false
summary: "Tìm hiểu WebSocket - cách tạo ứng dụng real-time như chat, notifications."
---

## WebSocket là gì?

WebSocket là giao thức cho phép communication hai chiều (bidirectional) giữa client và server. Khác với HTTP (request-response), WebSocket giữ connection mở để có thể gửi dữ liệu lúc bất kỳ.

### HTTP vs WebSocket

**HTTP:**
```
Client: GET /data
Server: Response
        (connection closed)
```

**WebSocket:**
```
Client: Upgrade to WebSocket
Server: Accept
        (persistent connection)
        
Server: Gửi data 1
Client: Nhận data 1

Client: Gửi message
Server: Nhận message

...connection stays open...
```

## WebSocket Handshake

### HTTP Upgrade Request

```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

### Server Response

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

**Status 101**: Switching Protocols - HTTP connection được upgrade thành WebSocket

## WebSocket Frame Format

WebSocket sử dụng frames để gửi dữ liệu:

```
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
+-+-+-+-+-------+-+-------------+
|F|R|R|R| opcode|M| Payload len |
|I|S|S|S|(4bits)|A|   (7 bits)  |
|N|V|V|V|       |S|             |
| |1|2|3|       |K|             |
+-+-+-+-+-------+-+-------------+
```

**FIN bit**: Frame cuối cùng?  
**Opcode**: 1 (Text), 2 (Binary), 8 (Close), 9 (Ping), 10 (Pong)  
**Mask**: Client-to-Server được mask, Server-to-Client không

## JavaScript WebSocket Client

### Tạo Connection

```javascript
// Tạo WebSocket connection
const ws = new WebSocket('ws://localhost:8080');

// Event: Connection mở
ws.onopen = (event) => {
    console.log('Connected to server');
    ws.send('Hello Server!');
};

// Event: Nhận message
ws.onmessage = (event) => {
    console.log('Message from server:', event.data);
};

// Event: Lỗi
ws.onerror = (event) => {
    console.error('WebSocket error:', event);
};

// Event: Connection đóng
ws.onclose = (event) => {
    console.log('Disconnected from server');
};
```

### Gửi dữ liệu

```javascript
// Send text
ws.send('Hello Server');

// Send JSON
ws.send(JSON.stringify({
    type: 'message',
    content: 'Hello!',
    timestamp: new Date()
}));

// Send binary data
const arrayBuffer = new ArrayBuffer(8);
ws.send(arrayBuffer);
```

### Reconnection Logic

```javascript
class WebSocketClient {
    constructor(url) {
        this.url = url;
        this.ws = null;
        this.reconnectAttempts = 0;
        this.maxReconnectAttempts = 5;
        this.reconnectDelay = 3000;
    }
    
    connect() {
        this.ws = new WebSocket(this.url);
        
        this.ws.onopen = () => {
            console.log('Connected');
            this.reconnectAttempts = 0;
        };
        
        this.ws.onmessage = (event) => {
            console.log('Message:', event.data);
        };
        
        this.ws.onclose = () => {
            console.log('Disconnected');
            this.reconnect();
        };
        
        this.ws.onerror = (error) => {
            console.error('Error:', error);
        };
    }
    
    reconnect() {
        if (this.reconnectAttempts < this.maxReconnectAttempts) {
            this.reconnectAttempts++;
            console.log(`Reconnecting... (${this.reconnectAttempts}/${this.maxReconnectAttempts})`);
            
            setTimeout(() => {
                this.connect();
            }, this.reconnectDelay);
        }
    }
    
    send(message) {
        if (this.ws.readyState === WebSocket.OPEN) {
            this.ws.send(message);
        }
    }
}

const client = new WebSocketClient('ws://localhost:8080');
client.connect();
```

## Java WebSocket Server

### Sử dụng javax.websocket

```java
import javax.websocket.*;
import javax.websocket.server.ServerEndpoint;

@ServerEndpoint("/chat")
public class ChatEndpoint {
    
    // Store sessions
    private static Set<Session> sessions = new CopyOnWriteArraySet<>();
    
    @OnOpen
    public void onOpen(Session session) {
        sessions.add(session);
        System.out.println("Client connected: " + session.getId());
        broadcast("Server: " + session.getId() + " joined");
    }
    
    @OnMessage
    public void onMessage(String message, Session session) {
        System.out.println("Message from " + session.getId() + ": " + message);
        broadcast(session.getId() + ": " + message);
    }
    
    @OnClose
    public void onClose(Session session) {
        sessions.remove(session);
        System.out.println("Client disconnected: " + session.getId());
        broadcast("Server: " + session.getId() + " left");
    }
    
    @OnError
    public void onError(Session session, Throwable error) {
        System.err.println("Error: " + error.getMessage());
    }
    
    private static void broadcast(String message) {
        for (Session session : sessions) {
            if (session.isOpen()) {
                try {
                    session.getBasicRemote().sendText(message);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

### Spring Boot WebSocket

```java
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.stereotype.Controller;

@Controller
public class ChatController {
    
    @MessageMapping("/chat")
    @SendTo("/topic/messages")
    public Message handleMessage(Message message) {
        System.out.println("Message: " + message.getContent());
        return message;
    }
}
```

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;

@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    
    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic");
        config.setApplicationDestinationPrefixes("/app");
    }
    
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws-chat").setAllowedOrigins("*");
    }
}
```

## Chat Application Example

### HTML

```html
<!DOCTYPE html>
<html>
<head>
    <title>WebSocket Chat</title>
</head>
<body>
    <input type="text" id="messageInput" placeholder="Type message...">
    <button onclick="sendMessage()">Send</button>
    <div id="messages"></div>
    
    <script>
        const ws = new WebSocket('ws://localhost:8080/chat');
        
        ws.onmessage = (event) => {
            const messagesDiv = document.getElementById('messages');
            const messageEl = document.createElement('div');
            messageEl.textContent = event.data;
            messagesDiv.appendChild(messageEl);
        };
        
        function sendMessage() {
            const input = document.getElementById('messageInput');
            ws.send(input.value);
            input.value = '';
        }
    </script>
</body>
</html>
```

## WebSocket vs Polling

### Polling (Inefficient)

```javascript
// Client constantly asks for updates
setInterval(() => {
    fetch('/api/messages')
        .then(r => r.json())
        .then(data => displayMessages(data));
}, 1000);  // Every 1 second
```

**Vấn đề:**
- Nhiều HTTP requests
- Latency cao
- Tốn bandwidth

### WebSocket (Efficient)

```javascript
// Server pushes updates whenever available
const ws = new WebSocket('ws://server/messages');

ws.onmessage = (event) => {
    displayMessages(JSON.parse(event.data));
};
```

**Ưu điểm:**
- Một connection
- Real-time
- Tiết kiệm bandwidth

## Use Cases

1. **Chat Applications**: Real-time messaging
2. **Notifications**: Live alerts, updates
3. **Live Updates**: Stock prices, sports scores
4. **Collaborative Tools**: Shared documents, whiteboards
5. **Games**: Multiplayer gaming
6. **Real-time Analytics**: Live dashboards

## WebSocket Challenges

### Connection Stability

```javascript
// Ping-Pong to keep connection alive
setInterval(() => {
    if (ws.readyState === WebSocket.OPEN) {
        ws.send(JSON.stringify({ type: 'ping' }));
    }
}, 30000);  // Every 30 seconds
```

### Load Balancing

```
Client1 -> Load Balancer -> Server A
Client2 -> Load Balancer -> Server B
           (Need to route messages between servers)
```

**Solution**: Use message brokers (RabbitMQ, Redis)

### Security

```javascript
// Secure WebSocket (WSS - encrypted)
const wss = new WebSocket('wss://localhost:8443');

// Add authentication
const ws = new WebSocket('ws://server?token=xyz123');
```

## Browser Support

WebSocket được hỗ trợ trên:
- Chrome, Firefox, Safari, Edge (Modern browsers)
- IE 10+ (Limited)

## Libraries

- **Socket.IO**: Real-time engine (fallback support)
- **SockJS**: WebSocket emulation layer
- **Stomp**: Message protocol over WebSocket

## Kết luận

WebSocket mang lại real-time capability cho web applications:

✅ Real-time communication  
✅ Lower latency  
✅ Bidirectional  
✅ Efficient  

Perfect cho chat, notifications, collaborative tools, và live updates!
