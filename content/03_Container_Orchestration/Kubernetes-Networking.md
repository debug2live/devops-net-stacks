# Kubernetes Networking

## k8s Siêu Sơ Đồ: Giải Phẫu & Truy Vết K8s Networking

```mermaid
sequenceDiagram
    autonumber
    participant App as 📱 Pod A (10.42.0.5)
    participant Kernel_A as 🧠 Host A Kernel
    participant flannel_A as 🚇 flannel.1 (A)
    participant eth0_A as 🌐 enp0s1 (A)

    participant eth0_B as 🌐 enp0s1 (B)
    participant flannel_B as 🚇 flannel.1 (B)
    participant Kernel_B as 🧠 Host B Kernel
    participant App_B as 🔵 Pod B (10.42.1.10)

    Note over App, App_B: MỤC TIÊU: Pod A gọi Pod B (10.42.1.10)

    rect rgb(240, 240, 240)
        Note over App, Kernel_A: BƯỚC 1: XÁC ĐỊNH ĐƯỜNG ĐI (Routing)
        App->>Kernel_A: Gửi gói tin qua veth/cni0
        Note right of Kernel_A: Lệnh check Route:<br/>ip route get 10.42.1.10
        Note right of Kernel_A: Kết quả mong đợi:<br/>10.42.1.10 dev flannel.1 src 10.42.0.1
    end

    rect rgb(220, 240, 220)
        Note over Kernel_A, flannel_A: BƯỚC 2: KIỂM TRA ĐẦU HẦM (VTEP)
        Kernel_A->>flannel_A: Chuyển gói tin vào flannel.1
        Note right of flannel_A: Lệnh check "Danh bạ" máy B:<br/>bridge fdb show dev flannel.1
        Note right of flannel_A: Kết quả mong đợi:<br/>MAC_B dst 192.168.2.13 (IP thật Host B)
    end

    rect rgb(255, 245, 230)
        Note over flannel_A, eth0_A: BƯỚC 3: QUAN SÁT SỰ BIẾN HÌNH (Encap)
        flannel_A->>eth0_A: Đóng gói VXLAN (UDP 4789)
        Note right of eth0_A: Lệnh bắt gói tin "đã đóng gói":<br/>sudo tcpdump -i enp0s1 udp port 4789 -n
    end

    eth0_A->>eth0_B: Bay qua mạng vật lý (Underlay)

    rect rgb(220, 240, 255)
        Note over eth0_B, flannel_B: BƯỚC 4: NHẬN HÀNG VÀ BÓC TÁCH (Decap)
        eth0_B->>flannel_B: Nhận gói UDP từ Host A
        Note left of flannel_B: Lệnh bắt gói tin "đã bóc vỏ":<br/>sudo tcpdump -i flannel.1 icmp -n
    end

    rect rgb(240, 240, 240)
        Note over flannel_B, App_B: BƯỚC 5: GIAO HÀNG CUỐI CÙNG
        flannel_B->>Kernel_B: Trả lại gói tin gốc (10.42.x.x)
        Kernel_B->>App_B: Đẩy vào cni0 đến Pod B
        Note left of App_B: Lệnh kiểm tra Pod nhận được chưa:<br/>kubectl exec pod-b -- tcpdump -i eth0
    end
```

### Chi tiết các vị trí kiểm tra (Pattern thực chiến)

Để đảm bảo thông tin là đúng, bạn hãy thực hiện kiểm tra theo thứ tự "Điểm kiểm soát" sau:

#### 1. Kiểm tra "Bản đồ" trên Host A (The Map)
Lệnh này giúp bạn biết Kernel định ném gói tin đi đâu ngay từ đầu.
- **Lệnh**: `ip route get 10.42.1.10`
- **Phân tích**:
  - Nếu nó hiện `dev flannel.1` ➔ Đúng, K8s đã học được đường sang máy B.
  - Nếu nó hiện `via 192.168.2.1` ➔ Sai, nó đang định ném ra Internet (K8s chưa học được mạng Overlay).

#### 2. Kiểm tra "Địa chỉ thật" của máy B (The Directory)
Làm sao `flannel.1` biết 192.168.2.13 là thằng giữ dải IP kia? Nó nằm trong bảng FDB (Forwarding Database).
- **Lệnh**: `bridge fdb show dev flannel.1`
- **Điểm cần soi**: Tìm dòng có chứa IP thật của máy B. Đây chính là "sợi dây" liên kết giữa mạng ảo (10.42.x.x) và mạng thật (192.168.x.x).

#### 3. Kiểm tra "Đóng gói" trên đường truyền (The Tunnel)
Dùng lệnh này trên card mạng thật để thấy gói tin "biến hình":
- **Lệnh**: `sudo tcpdump -i enp0s1 udp port 4789 -vv -X`
- **Giải mã**: Flag `-X` sẽ giúp bạn nhìn thấy nội dung gói tin. Bạn sẽ thấy bên ngoài là IP của Host A/B, nhưng sâu bên trong đống mã HEX là IP của Pod A/B.

#### 4. Kiểm tra "Láng giềng" (Neighbor)
K8s cần biết địa chỉ MAC của các interface ảo.
- **Lệnh**: `ip neighbor show dev flannel.1`
- **Ý nghĩa**: Nếu dòng này trống, nghĩa là máy A chưa bao giờ "nói chuyện" được với `flannel.1` của máy B.

---

🏁 **Tổng kết Pattern kiểm tra nhanh**:
- Chưa ping được? ➔ Check `ip route` (Bản đồ có sai không?).
- Route đúng nhưng vẫn timeout? ➔ Check `bridge fdb` (Có biết IP thật của máy kia không?).
- FDB đúng? ➔ Check `tcpdump -i enp0s1` (Gói tin có thực sự bay ra khỏi card mạng không?).

## k8s ClusterIP Service: Luồng dữ liệu và NAT

```mermaid
sequenceDiagram
    autonumber
    participant PodA as 📱 Pod A (Frontend)<br/>IP: 10.42.0.7
    box rgb(240, 248, 255) KERNEL MÁY HOST (Nơi diễn ra "ảo thuật")
    participant Iptables as 🛡️ iptables (Data Plane)<br/>(Kẻ thực thi)
    participant Conntrack as 📖 Conntrack Table<br/>(Sổ ghi nhớ)
    end
    participant KProxy as ⚙️ Kube-proxy (Control Plane)<br/>(Người quản lý)
    box rgb(255, 245, 238) ĐỐI TƯỢNG K8S (Database)
    participant SvcObj as 📛 Service Object<br/>VIP: 10.43.0.1 (ẢO)
    participant EPObj as 📋 Endpoints Object<br/>Thật: [10.42.1.10, 10.42.2.20]
    end
    participant PodB as 🔵 Pod B1 (Backend)<br/>IP: 10.42.1.10

    Note over KProxy, EPObj: GIAI ĐOẠN 1: CẬP NHẬT DANH BẠ (NGẦM)
    KProxy->>SvcObj: Watch Service (Biết VIP 10.43.0.1)
    KProxy->>EPObj: Watch Endpoints (Lấy list IP THẬT)
    Note right of KProxy: [Danh bạ nội bộ]<br/>10.43.0.1 -> [10.42.1.10, 10.42.2.20]
    KProxy->>Iptables: Nạp luật NAT vào Kernel:<br/>"Gặp Dst 10.43.0.1 -> DNAT sang 10.42.1.10 (50%) hoặc 10.42.2.20 (50%)"

    Note over PodA, PodB: GIAI ĐOẠN 2: LUỒNG DỮ LIỆU (THỰC TẾ)

    PodA->>Iptables: Gửi request tới Service IP
    Note right of PodA: [GÓI GỐC]<br/>Src: 10.42.0.7<br/>Dst: 10.43.0.1 (ẢO)

    rect rgb(255, 228, 225)
    Note over Iptables: BƯỚC 2.1: IPTABLES "BẺ LÁI" (DNAT)
    Iptables->>Iptables: Khớp luật: Destination = 10.43.0.1
    Iptables->>Iptables: Chọn ngẫu nhiên Endpoint: 10.42.1.10
    Note right of Iptables: [HÀNH ĐỘNG PHẪU THUẬT]<br/>Xóa Dst: 10.43.0.1 (ẢO)<br/>Ghi đè Dst: 10.42.1.10 (THẬT)
    end

    Iptables->>Conntrack: Ghi lại trạng thái kết nối
    Note right of Conntrack: [SỔ GHI NHỚ]<br/>Mới: Src 10.42.0.7 | Dst 10.43.0.1<br/>Đã NAT thành: Dst 10.42.1.10

    Iptables->>PodB: Gửi gói tin ĐÃ ĐƯỢC NAT đi tiếp
    Note right of Iptables: [GÓI SAU NAT]<br/>Src: 10.42.0.7<br/>Dst: 10.42.1.10 (THẬT)

    PodB->>PodB: Xử lý request...
    PodB->>Iptables: Gửi gói tin trả lời (Response)
    Note left of PodB: [GÓI TRẢ LỜI]<br/>Src: 10.42.1.10 (THẬT)<br/>Dst: 10.42.0.7

    rect rgb(224, 255, 255)
    Note over Iptables: BƯỚC 2.2: IPTABLES "PHỤC HỒI" (Un-NAT)
    Iptables->>Conntrack: Tra sổ: "Gói từ 10.42.1.10 về 10.42.0.7 là của ai?"
    Conntrack-->>Iptables: "Của kết nối Src 10.42.0.7 | Dst 10.43.0.1 ban đầu"
    Note right of Iptables: [HÀNH ĐỘNG HỒI PHỤC]<br/>Xóa Src: 10.42.1.10 (THẬT)<br/>Ghi đè Src: 10.43.0.1 (ẢO)
    end

    Iptables->>PodA: Trả thư về cho Pod A
    Note left of PodA: [GÓI NHẬN ĐƯỢC]<br/>Src: 10.43.0.1 (Đúng ý Pod A)<br/>Dst: 10.42.0.5
```

### 🔍 Chú thích chi tiết (Visualized Key Points)
Hãy nhìn vào sơ đồ và đối chiếu với các điểm sau để thấy rõ "bản chất" của chúng:

#### 1. Thể hiện rõ "Service IP (VIP) là 10.43.0.1"
- **Trên sơ đồ**: Nhìn vào đối tượng 📛 Service Object (VIP: 10.43.0.1).
- **Đặc điểm**: Nó nằm trong box "ĐỐI TƯỢNG K8S (Database)". Nghĩa là nó chỉ là một dòng dữ liệu cấu hình.
- **Bằng chứng "ẢO"**: Nhìn vào Gói gốc (Bước 3) và Gói sau NAT (Bước 7). IP 10.43.0.1 biến mất ngay khi chạm vào Kernel (Bước 4). Nó không bao giờ bay ra khỏi card mạng thật.

#### 2. Thể hiện rõ "Endpoints là gì?"
- **Trên sơ đồ**: Nhìn vào đối tượng 📋 Endpoints Object (Thật: [10.42.1.10, 10.42.2.20]).
- **Bản chất**: Đây chính là cái "Danh bạ IP thật". Kube-proxy (Bước 1 & 2) lấy dữ liệu từ đây để biết đường mà nạp luật vào iptables.
- **Bằng chứng "THẬT"**: Nhìn vào Gói sau NAT (Bước 7). Địa chỉ Destination đã trở thành 10.42.1.10. Đây là IP của một card mạng thật, một Pod thật đang chạy.

#### 3. Vai trò của iptables (Kẻ thực thi "phẫu thuật")
- **Trên sơ đồ**: Box "BƯỚC NHẢY IPTABLES" (Bước 4-6) hoặc BƯỚC 2.1: IPTABLES "BẺ LÁI".
- **Hành động**: Nó thực hiện việc xóa IP ảo và ghi đè IP thật trực tiếp lên gói tin. Đây không phải là route, đây là Destination NAT (DNAT).

#### 4. Vai trò của Conntrack (Cuốn sổ ghi nhớ đường về)
- **Trên sơ đồ**: Box "BƯỚC HỒI PHỤC" (Bước 11-13) hoặc BƯỚC 2.2: IPTABLES "PHỤC HỒI".
- **Tác dụng**: Nếu không có bước tra sổ (Bước 11) và phục hồi (Bước 12), Pod A sẽ nhận được thư từ 10.42.1.10 và nó sẽ từ chối nhận vì nó đang chờ phản hồi từ 10.43.0.1. Conntrack giúp "đóng kịch" cho đến phút cuối cùng.

### 🛠️ Cách để bạn "thấy" sự biến mất của Service IP trên thực tế:
Bạn hãy dùng tcpdump trên host-a để bắt gói tin khi đang curl đến Service IP:

1. **Bắt gói tin trên card veth của Pod A (Trước NAT)**:
   ```bash
   tcpdump -i vethxxxx host 10.43.0.1
   ```
   ➔ Bạn sẽ thấy IP đích là 10.43.0.1.

2. **Bắt gói tin trên card enp0s1 (Sau NAT)**:
   ```bash
   tcpdump -i enp0s1 host 10.43.0.1
   ```
   ➔ Bạn sẽ KHÔNG thấy gói tin nào. Vì lúc này gói tin đã mang IP đích của Pod Backend (10.42.1.10).
