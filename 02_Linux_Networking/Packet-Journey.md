# Workflow: Hành trình của Gói tin (Packet Journey)

Để một gói tin từ một vùng cô lập (Namespace) đi ra được Internet, nó phải vượt qua 5 trạm kiểm soát chính. Nếu bất kỳ trạm nào gặp sự cố, gói tin sẽ bị "kẹt" lại ngay lập tức.

## 🏗️ Kiến trúc Mô phỏng (Packet Flow Architecture)

Dưới đây là sơ đồ trình tự thể hiện đường đi của gói tin:

```mermaid
sequenceDiagram
    participant App as 🔴 Namespace (App)
    participant Veth as 🔗 Veth Pair
    participant Bridge as 🌉 Linux Bridge (br0)
    participant Host as 💻 Linux Host (Kernel)
    participant ISP as 🌐 Internet (Google)

    Note over App: Bước 1: Tra cứu Route Table
    App->>Veth: Gửi tới Default Gateway (10.0.0.254)
    Veth->>Bridge: Chui qua dây veth
    Note over Bridge: Bước 2: Layer 2 Switching
    Bridge->>Host: Đưa gói tin lên Interface br0
    Note over Host: Bước 3: IP Forwarding (L3)
    Note over Host: Bước 4: NAT / Masquerade
    Host->>ISP: Đẩy ra card eth0 (với IP thật)
    ISP-->>Host: Trả lời (Reply)
    Host-->>App: Trả về Namespace (Un-NAT)
```
