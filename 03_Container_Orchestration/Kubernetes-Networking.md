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
