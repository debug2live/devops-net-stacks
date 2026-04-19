# Kubelet Architecture

## Sequence Diagram

```mermaid
sequenceDiagram
    autonumber
    participant API as 🚪 kube-apiserver

    box rgb(255, 250, 230) WORKER NODE
    participant Kube as 👷 Kubelet
    participant PStat as 📊 Pod Status Manager
    participant Runtime as 📦 Container Runtime (CRI)
    end

    API->>Kube: 1. Giao PodSpec (Bản thiết kế chi tiết)

    Note over Kube, Runtime: GIAI ĐOẠN THỰC THI (CRI)
    Kube->>Runtime: 2. "CRI ơi, tạo Sandbox và chạy Container này!"
    Runtime-->>Kube: 3. "Đã xong, Container đang Running."

    Note over Kube, PStat: GIAI ĐOẠN GIÁM SÁT
    loop Định kỳ mỗi vài giây
        Kube->>Runtime: 4. Liveness Probe (Anh còn sống không?)
        Runtime-->>Kube: 5. "Vẫn sống khỏe!"
        Kube->>PStat: 6. Cập nhật trạng thái Pod
    end

    Note over Kube, API: GIAI ĐOẠN BÁO CÁO
    PStat->>API: 7. Báo cáo Status: "Running"
    Kube->>API: 8. Node Status: "Ready" (Heartbeat)
```

## Chức năng của Kubelet

Hãy tưởng tượng Kubelet như một người quản đốc tại công trường (Node):

- **Tiếp nhận bản thiết kế:** Nó không nhìn vào YAML của bạn, nó nhìn vào các PodSpec mà API Server gửi tới.
- **Điều hành động cơ:** Nó ra lệnh cho Container Runtime (Containerd) chạy hoặc dừng Container.
- **Giám sát sức khỏe:** Nó kiểm tra xem Container có còn sống không. Nếu "ngỏm", nó sẽ tự động khởi động lại (Restart Policy).
- **Báo cáo thành tích:** Định kỳ, nó gửi "nhịp tim" (Heartbeat) về Master để báo: "Tôi vẫn khỏe, các Pod vẫn chạy tốt".
