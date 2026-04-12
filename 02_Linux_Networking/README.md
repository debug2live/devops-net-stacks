# Level 2: Linux Networking - Deep Dive Documentation

Chào mừng bạn đến với The DevOps Playground. Ở Level này, chúng ta sẽ tự tay xây dựng một môi trường mạng cô lập bằng Linux Network Namespace và mô phỏng lại chính xác cách Docker hoạt động dưới "nắp capo".

## 📝 Tài liệu & Thực hành

- [**1. Tổng quan Kiến trúc**](./Architecture.md): Sơ đồ thiết kế hệ thống mạng ảo sử dụng Namespace, Bridge và Veth Pair.
- [**2. Quy trình thiết lập**](./Setup-Steps.md): Các lệnh bash từng bước để khởi tạo môi trường (Namespace, Bridge, Veth, IP Routing).
- [**3. Các bước kiểm chứng**](./Testing-and-Verification.md): Hướng dẫn sử dụng `ping` và `tcpdump` để kiểm tra luồng đi của gói tin.
- [**4. Hành trình của Gói tin (Workflow)**](./Packet-Journey.md): Sơ đồ trình tự mô tả 5 trạm kiểm soát mà một gói tin phải vượt qua để ra Internet.
- [**5. Nhật ký Sửa lỗi & Cheat Sheet**](./Troubleshooting.md): Cẩm nang xử lý các lỗi thường gặp (Routing, NAT, IP Forwarding) theo từng trạm kiểm soát và bài học rút ra.
