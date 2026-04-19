# Kubernetes Architecture

## Sequence Diagram

```mermaid
sequenceDiagram
    autonumber
    actor Dev as 🧑‍💻 DevOps Engineer

    box rgb(230, 242, 255) 🧠 CONTROL PLANE (Master Node - Vùng Đầu Não)
    participant API as 🚪 API Server (Cổng vào)
    participant ETCD as 📖 etcd (Trí nhớ)
    participant SCHED as 📅 Scheduler (Điều phối)
    participant CTRL as 🎮 Controller Manager (Giám sát)
    end

    box rgb(255, 250, 230) 🏗️ WORKER NODE (Slave Node - Vùng Cơ Bắp)
    participant Kubelet as 👷 Kubelet (Đội trưởng)
    participant CR as 📦 Container Runtime (Động cơ)
    participant Proxy as 👮 kube-proxy (Cảnh sát mạng)
    end

    Note over Dev, API: GIAI ĐOẠN 1: TIẾP NHẬN
    Dev->>API: 1. kubectl apply -f nginx.yaml
    API->>ETCD: 2. Ghi bản thiết kế vào "Trí nhớ"

    Note over API, CTRL: GIAI ĐOẠN 2: GIÁM SÁT
    CTRL->>API: 3. Kiểm tra: "Thực tế có Nginx chưa?"
    CTRL-->>API: "Chưa có! Hãy tạo mới đi."

    Note over API, SCHED: GIAI ĐOẠN 3: ĐIỀU PHỐI (CHỌN NODE)
    SCHED->>API: 4. Xem tài nguyên: "Host A còn trống nhiều RAM"
    API->>ETCD: 5. Ghi chú: "Nginx này sẽ chạy ở Host A"

    Note over API, Kubelet: GIAI ĐOẠN 4: THỰC THI (CẬP NHẬT NODE)
    Kubelet->>API: 6. "Có việc gì cho Host A của tôi không?"
    API-->>Kubelet: 7. "Lấy bản thiết kế Nginx này về chạy ngay!"

    Kubelet->>CR: 8. "Động cơ ơi, kéo Image và nổ máy!"
    CR-->>Kubelet: 9. "Container đã sẵn sàng!"

    Note over API, Proxy: GIAI ĐOẠN 5: KẾT NỐI MẠNG
    Proxy->>API: 10. "Có cần mở cổng cho Nginx không?"
    Proxy->>Proxy: 11. Cài đặt iptables (Mạch máu Networking)
```

## Giải thích các thành phần

| Thành phần | Tên kỹ thuật | Chức năng (Dễ hiểu) |
| :--- | :--- | :--- |
| Cái miệng & Tai | kube-apiserver | Tiếp nhận lệnh từ bạn (kubectl) và là trung tâm liên lạc giữa các bộ phận. |
| Trí nhớ (DB) | etcd | Cuốn sổ cái lưu trạng thái của toàn bộ hệ thống. Chỉ API Server mới được ghi vào đây. |
| Người điều phối | kube-scheduler | Tính toán: "Pod này nên nằm ở Node nào thì hợp lý nhất?". |
| Người giám sát | kube-controller-manager | Đảm bảo thực tế luôn khớp với mong muốn (Ví dụ: Chết 1 Pod là ra lệnh tạo lại ngay). |
| Đội trưởng Node | kubelet | "Đại sứ" của Master trên mỗi Node, quản lý trực tiếp vòng đời của Container. |
| Cảnh sát GT | kube-proxy | Quản lý mạng, cài đặt iptables (thứ chúng ta đã học rất kỹ). |
| Động cơ | Container Runtime | Phần mềm thực sự để chạy container (thường là containerd). |
