# Kubernetes LoadBalancer Networking

```mermaid
sequenceDiagram
    autonumber
    participant User as 💻 Người dùng (Browser)
    participant VIP as 🌐 Virtual IP (LoadBalancer)<br/>192.168.2.100

    box rgb(240, 248, 255) TẦNG ĐIỀU PHỐI (LoadBalancer)
    participant Speaker as 📢 MetalLB / ARP<br/>(Chiếm quyền IP)
    end

    box rgb(225, 245, 255) TẦNG TIẾP NHẬN (NodePort)
    participant HostA as 🖥️ Host A (192.168.2.12)
    participant HostB as 🖥️ Host B (192.168.2.13)
    end

    participant Pod as 🔵 Pod Backend

    Note over User, VIP: NGƯỜI DÙNG GỌI VÀO IP DUY NHẤT
    User->>VIP: Truy cập http://192.168.2.100 (Port 80)

    Note over VIP, Speaker: PHÉP THUẬT 1: ARP ADVERTISING
    Speaker-->>User: "Ta là 192.168.2.100, hãy gửi gói tin về Host A!"

    Note over User, HostA: CHẶNG 1: GÓI TIN ĐẾN HOST ĐANG GIỮ VIP
    User->>HostA: Gửi gói tin mang Dst IP 192.168.2.100

    Note over HostA, Pod: CHẶNG 2: QUAY LẠI WORKFLOW NODEPORT
    HostA->>HostA: iptables: "192.168.2.100:80 -> Chuyển vào Service"
    HostA->>HostA: iptables: "Chuyển tiếp vào ClusterIP -> Pod"
    HostA->>Pod: Đi qua Flannel (nếu cần) và đến đích
```

### 3 Điểm "Bản chất" của LoadBalancer locally:
1. **Sự "Chiếm hữu" IP (ARP/BGP)**
Tại sao bạn gõ IP 192.168.2.100 mà gói tin lại biết chui vào Host A?

Bản chất: Trong mạng nội bộ, các máy tìm nhau qua địa chỉ MAC. Một phần mềm (như MetalLB) sẽ đứng ra nói dối với cả mạng LAN rằng: "Địa chỉ MAC của Host A hiện tại cũng chính là địa chỉ của IP 192.168.2.100".

Khi đó, toàn bộ traffic đổ dồn về Host A.

2. **LoadBalancer là một "vỏ bọc" của NodePort**
Khi bạn dùng dịch vụ Cloud, thực chất Cloud Provider tạo ra một con LoadBalancer vật lý/phần mềm nằm ngoài Cluster. Con đó sẽ check: "Node A còn sống không? Node B còn sống không?". Sau đó nó mới forward traffic vào các NodePort tương ứng.

3. **Sự khác biệt Cloud vs Local**
Local (MetalLB): Một Node trong Cluster sẽ "đóng vai" LoadBalancer bằng cách mượn thêm một IP phụ.

Cloud (NLB/ALB): Một thiết bị chuyên dụng của Amazon/Google sẽ đứng ngoài hoàn toàn, tách biệt với tài nguyên tính toán của các Node.
