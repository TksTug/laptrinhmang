# Socket Programming với Java — Giao tiếp Mạng Cơ bản


## Giới thiệu Socket Programming

Socket là một điểm cuối của một kết nối mạng. Thông qua socket, hai chương trình có thể giao tiếp với nhau qua mạng, dù chúng ở trên cùng một máy tính hay trên các máy tính khác nhau.

### Socket là gì?

- **Socket = Ổ cắm mạng**: Tương tự như ổ cắm điện, socket là nơi kết nối vào mạng.
- **IP Address + Port**: Một socket được định danh bằng địa chỉ IP và số cổng (port).
- **Hai loại Socket**: TCP Socket (định hướng kết nối) và UDP Socket (không kết nối).

## TCP Socket Programming

TCP (Transmission Control Protocol) đảm bảo dữ liệu được gửi đến đích một cách chính xác và theo thứ tự.

### 1. Tạo TCP Server

```java
import java.io.*;
import java.net.*;

public class TCPServer {
    public static void main(String[] args) {
        try {
            // Tạo ServerSocket lắng nghe trên cổng 5000
            ServerSocket serverSocket = new ServerSocket(5000);
            System.out.println("Server đang lắng nghe trên cổng 5000...");
            
            // Chấp nhận kết nối từ client
            Socket clientSocket = serverSocket.accept();
            System.out.println("Client đã kết nối từ: " + clientSocket.getInetAddress());
            
            // Tạo input stream để nhận dữ liệu
            BufferedReader input = new BufferedReader(
                new InputStreamReader(clientSocket.getInputStream())
            );
            
            // Tạo output stream để gửi dữ liệu
            PrintWriter output = new PrintWriter(
                clientSocket.getOutputStream(), true
            );
            
            // Đọc dữ liệu từ client
            String message = input.readLine();
            System.out.println("Nhận được: " + message);
            
            // Gửi phản hồi
            output.println("Server đã nhận: " + message);
            
            // Đóng kết nối
            clientSocket.close();
            serverSocket.close();
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 2. Tạo TCP Client

```java
import java.io.*;
import java.net.*;

public class TCPClient {
    public static void main(String[] args) {
        try {
            // Kết nối đến server
            Socket socket = new Socket("localhost", 5000);
            System.out.println("Đã kết nối đến server");
            
            // Tạo output stream để gửi dữ liệu
            PrintWriter output = new PrintWriter(
                socket.getOutputStream(), true
            );
            
            // Tạo input stream để nhận dữ liệu
            BufferedReader input = new BufferedReader(
                new InputStreamReader(socket.getInputStream())
            );
            
            // Gửi dữ liệu
            output.println("Xin chào Server!");
            
            // Nhận phản hồi
            String response = input.readLine();
            System.out.println("Phản hồi từ server: " + response);
            
            // Đóng kết nối
            socket.close();
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## UDP Socket Programming

UDP (User Datagram Protocol) không đảm bảo gửi đến mà chỉ cố gắng gửi dữ liệu nhanh nhất có thể.

### UDP Server

```java
import java.net.*;

public class UDPServer {
    public static void main(String[] args) {
        try {
            // Tạo DatagramSocket lắng nghe trên cổng 5001
            DatagramSocket socket = new DatagramSocket(5001);
            byte[] receiveData = new byte[1024];
            
            System.out.println("UDP Server đang lắng nghe trên cổng 5001...");
            
            // Nhận datagram
            DatagramPacket receivePacket = new DatagramPacket(
                receiveData, receiveData.length
            );
            socket.receive(receivePacket);
            
            // Lấy dữ liệu
            String message = new String(receivePacket.getData(), 0, receivePacket.getLength());
            System.out.println("Nhận được: " + message);
            System.out.println("Từ: " + receivePacket.getAddress());
            
            // Gửi phản hồi
            byte[] sendData = "Đã nhận dữ liệu".getBytes();
            DatagramPacket sendPacket = new DatagramPacket(
                sendData,
                sendData.length,
                receivePacket.getAddress(),
                receivePacket.getPort()
            );
            socket.send(sendPacket);
            
            socket.close();
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### UDP Client

```java
import java.net.*;

public class UDPClient {
    public static void main(String[] args) {
        try {
            DatagramSocket socket = new DatagramSocket();
            byte[] sendData = "Xin chào UDP Server!".getBytes();
            
            // Gửi datagram đến server
            DatagramPacket sendPacket = new DatagramPacket(
                sendData,
                sendData.length,
                InetAddress.getByName("localhost"),
                5001
            );
            socket.send(sendPacket);
            
            // Nhận phản hồi
            byte[] receiveData = new byte[1024];
            DatagramPacket receivePacket = new DatagramPacket(
                receiveData, receiveData.length
            );
            socket.receive(receivePacket);
            
            String response = new String(
                receivePacket.getData(), 0, receivePacket.getLength()
            );
            System.out.println("Phản hồi: " + response);
            
            socket.close();
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## Multi-threaded Server

Để xử lý nhiều client cùng lúc, sử dụng threads:

```java
import java.io.*;
import java.net.*;

public class MultiThreadedServer {
    public static void main(String[] args) {
        try {
            ServerSocket serverSocket = new ServerSocket(5000);
            System.out.println("Server đang chạy...");
            
            int clientNumber = 0;
            while (true) {
                Socket clientSocket = serverSocket.accept();
                clientNumber++;
                System.out.println("Client #" + clientNumber + " đã kết nối");
                
                // Tạo thread mới để xử lý client
                Thread clientThread = new Thread(
                    new ClientHandler(clientSocket, clientNumber)
                );
                clientThread.start();
            }
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

class ClientHandler implements Runnable {
    private Socket socket;
    private int clientNumber;
    
    public ClientHandler(Socket socket, int clientNumber) {
        this.socket = socket;
        this.clientNumber = clientNumber;
    }
    
    public void run() {
        try {
            BufferedReader input = new BufferedReader(
                new InputStreamReader(socket.getInputStream())
            );
            PrintWriter output = new PrintWriter(socket.getOutputStream(), true);
            
            String message;
            while ((message = input.readLine()) != null) {
                System.out.println("Client #" + clientNumber + ": " + message);
                output.println("Echo: " + message);
            }
            
            socket.close();
            System.out.println("Client #" + clientNumber + " đã ngắt kết nối");
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Khái niệm quan trọng

### Port Numbers
- **0-1023**: Hệ thống (System)
- **1024-49151**: Người dùng đăng ký
- **49152-65535**: Động/Riêng tư

### Connection States
- **LISTENING**: Server chờ kết nối
- **ESTABLISHED**: Kết nối đã thiết lập
- **CLOSING**: Đóng kết nối

### Timeout
```java
socket.setSoTimeout(5000); // 5 giây timeout
```

## Kết luận

Socket Programming là nền tảng của giao tiếp mạng. Hiểu rõ TCP/UDP sockets sẽ giúp bạn xây dựng các ứng dụng mạng mạnh mẽ như chat, game, hoặc web services.

