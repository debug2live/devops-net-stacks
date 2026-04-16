# Docker Networking: Single Host

## Sơ đồ luồng: Container đơn lẻ ra Internet (Single Host)

```mermaid
sequenceDiagram
    autonumber
    participant App as 📱 App (Container IP: 172.17.0.2)
    participant NS as 🔴 Namespace Container
    participant docker0 as 🌉 Bridge docker0 (172.17.0.1)
    participant Kernel as 🧠 Linux Kernel (Routing & NAT)
    participant eth0 as 🌐 Card mạng thật (192.168.2.12)
    participant Router as 🛤️ Gateway/Internet (192.168.2.1)

    Note over App, Router: MỤC TIÊU: Container gọi Google (8.8.8.8)

    rect rgb(240, 240, 240)
    Note over App, docker0: BƯỚC 1: RỜI KHỎI NHÀ (Layer 2)
    App->>NS: ping 8.8.8.8
    Note right of NS: Kiểm tra IP đích:<br/>8.8.8.8 không thuộc 172.17.0.0/16
    NS->>docker0: Đẩy ra Default Gateway (172.17.0.1)
    Note right of docker0: Lệnh check veth pair:<br/>ip link show master docker0
    end

    rect rgb(220, 240, 220)
    Note over docker0, Kernel: BƯỚC 2: QUYẾT ĐỊNH HƯỚNG ĐI (Layer 3)
    docker0->>Kernel: Chuyển gói tin lên bộ não điều hướng
    Note right of Kernel: Lệnh check Routing:<br/>ip route get 8.8.8.8
    Note right of Kernel: Kết quả mong đợi:<br/>8.8.8.8 via 192.168.2.1 dev eth0
    Note right of Kernel: ĐIỀU KIỆN CẦN:<br/>cat /proc/sys/net/ipv4/ip_forward = 1
    end

    rect rgb(255, 245, 230)
    Note over Kernel, eth0: BƯỚC 3: THAY TÊN ĐỔI HỌ (NAT - POSTROUTING)
    Kernel->>eth0: Chuẩn bị đẩy gói tin ra card vật lý
    Note right of eth0: Lệnh check Rule NAT:<br/>sudo iptables -t nat -L POSTROUTING -n
    Note right of eth0: HÀNH ĐỘNG MASQUERADE:<br/>Sửa Src IP: 172.17.0.2 -> 192.168.2.12
    end

    rect rgb(220, 240, 255)
    Note over eth0, Router: BƯỚC 4: RA THẾ GIỚI
    eth0->>Router: Gói tin đã NAT bay tới Router
    Note left of Router: Lệnh bắt gói tin đã "ngụy trang":<br/>sudo tcpdump -i eth0 dst 8.8.8.8 -n
    Note left of Router: Lúc này Google chỉ thấy IP máy Host!
    end
```

## 🔍 Các "Chốt chặn" và cách kiểm tra chi tiết:

### 1. Kiểm tra "Dây cáp ảo" (Veth Pair)
Làm sao biết Container đã nối đúng vào Bridge docker0 chưa?

**Lệnh:**
```bash
brctl show docker0
# hoặc
ip link show master docker0
```

**Xác nhận:** Nếu bạn thấy một interface dạng `vethxxxx`, nghĩa là "dây" đã cắm. Nếu không thấy, container bị cô lập hoàn toàn.

### 2. Kiểm tra "Quyền chuyển tiếp" (The Forwarding Key)
Dù bảng route đúng, nhưng nếu Kernel không cho phép gói tin nhảy từ card này sang card khác, nó sẽ bị drop.

**Lệnh:**
```bash
sysctl net.ipv4.ip_forward
```

**Xác nhận:** Phải bằng `1`. Đây chính là cái "chốt cửa" giữa `docker0` và `eth0`.

### 3. Kiểm tra "Tấm mặt nạ" (NAT/Masquerade)
Tại sao phải NAT? Vì Internet không biết `172.17.x.x` là ai. Máy Host phải đứng ra "đại diện".

**Lệnh:**
```bash
sudo iptables -t nat -S POSTROUTING
```

**Soi dòng chữ:** `-A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE`

**Ý nghĩa:** "Mọi thứ từ dải Docker đi ra ngoài (không phải quay lại docker0) thì hãy dán nhãn IP của máy Host vào".

### 4. Mô phỏng Route (Pattern 3C)
Khi bạn gõ `ip route get 8.8.8.8`:

- **Cái gì (Dest):** `8.8.8.8`
- **Cổng nào (Dev):** `eth0` (Card thật)
- **Của ai (Via):** `192.168.2.1` (Router nhà mạng)
