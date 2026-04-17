# Level 1: Core Foundations

Chào mừng bạn đến với phần "Móng nhà". Ở Level này, chúng ta sẽ đi sâu vào nền tảng của mạng máy tính, đặc biệt là cách mà một thiết bị giao tiếp với thế giới bên ngoài.

## 📝 Cấu trúc "Hành trang" của bạn

Thư mục này bao gồm các tài liệu thực chiến giúp bạn hiểu từ lý thuyết đến thực hành các lệnh cơ bản nhất trên Terminal của mình:

- [**Architecture-Overview.md**](./Architecture-Overview.md): Chứa sơ đồ Mermaid và giải thích dòng chảy gói tin (từ máy tính -> mạng nội bộ -> Internet).
- [**IP-and-Subnetting.md**](./IP-and-Subnetting.md): Tổng hợp các khái niệm cốt lõi (IP, MAC, Subnet, DNS) và cách phân biệt Private/Public IP trên máy mình.
- [**Interface-Investigation.md**](./Interface-Investigation.md): Các ví dụ và lệnh "gối đầu giường" như kiểm tra sức khỏe card mạng (`ifconfig`, `netstat -i`).
- [**Routing-Table.md**](./Routing-Table.md): Giải thích về Default Gateway và cách máy bạn chọn đường đi (`netstat -rn`).

---

## 💡 Lời Kết Level 1

Bạn đã hoàn thành phần "móng nhà". Từ một Backend Developer chỉ biết code gọi API, giờ bạn đã hiểu:
- Code của bạn thực tế đi qua card mạng nào (ví dụ: `en7`).
- Nó dùng địa chỉ gì để "nói chuyện" với Router (ví dụ: `192.168.1.12`).
- Nó thoát ra thế giới bằng danh tính nào (Public IP).
- Nó đi ra bằng con đường nào (Default Route).
