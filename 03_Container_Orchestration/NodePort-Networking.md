# Kubernetes NodePort Networking

## k8s NodePort Service: Luồng dữ liệu và NAT

```mermaid
sequenceDiagram
    autonumber
    participant User as 💻 Bên ngoài (Browser)

    box rgb(240, 248, 255) TẦNG TIẾP NHẬN (NodePort)
    participant Port as 🚪 Host Port (30001)
    participant NAT_NP as 🛡️ iptables (NodePort Rule)
    end

    box rgb(225, 245, 255) TẦNG LOGIC CŨ (ClusterIP)
    participant NAT_Svc as 🛡️ iptables (Service Rule)
    participant Route as 🧠 Kernel Routing
    participant Flannel as 🚇 flannel.1
    end

    participant PodB as 🔵 Pod B (Máy khác)

    User->>Port: Gọi IP_Host:30001

    Note over Port, NAT_NP: ĐÂY LÀ PHẦN MỚI CỦA NODEPORT
    Port->>NAT_NP: Gói tin chạm cổng vật lý
    NAT_NP->>NAT_NP: "Lệnh bài: 30001 -> Chuyển vào Service X"

    Note over NAT_NP, NAT_Svc: QUAY VỀ WORKFLOW CŨ (BÀI TOÁN 1)
    NAT_NP->>NAT_Svc: Đẩy gói tin vào chuỗi xử lý Service
    NAT_Svc->>NAT_Svc: DNAT: Đổi Dst thành IP Pod B (10.42.1.10)

    Note over NAT_Svc, Flannel: QUAY VỀ WORKFLOW CÔNG TRÌNH (FLANNEL)
    NAT_Svc->>Route: Chuyển gói đã NAT cho Route
    Route->>Flannel: "Địa chỉ 10.42.1.x hả? Vào hầm flannel ngay!"
    Flannel->>PodB: Đóng gói VXLAN và bay đi...
```

### 3 Điểm "Chốt" để bạn không bao giờ nhầm lẫn:

1. **Tính đồng nhất**: NodePort mở trên tất cả các node vì kube-proxy chạy trên tất cả các node. Nó nạp cùng một bộ quy tắc iptables giống hệt nhau vào mọi máy Host.

2. **Sự kế thừa**: K8s không viết lại logic mới cho NodePort. Nó chỉ tạo ra một quy tắc iptables ở tầng trên cùng (Chain KUBE-NODEPORTS) để "nhảy" vào quy tắc của Service (Chain KUBE-SERVICES).

3. **Điểm mù (Sự lãng phí)**: Đây là điểm bạn cần lưu ý: Nếu bạn gọi vào Host A, nhưng Pod lại ở Host B, gói tin phải đi vòng: User ➔ Host A ➔ Host B. Điều này làm tăng độ trễ (latency).
