# TỔNG HỢP CÁC KHÁI NIỆM CỐT LÕI (CHECKLIST)

| Khái niệm | Giải thích thực chiến | Cách nhận biết nhanh |
| :--- | :--- | :--- |
| **IP Address** | "Số nhà" để định vị thiết bị. | `ifconfig` (Private) / `curl ifconfig.me` (Public) |
| **MAC Address** | "Số khung" vật lý của card mạng (duy nhất). | Dòng `ether` trong `ifconfig`. |
| **Subnet Mask** | Xác định "phường/quận" (phạm vi mạng). | `/24` (254 máy), `/16` (65k máy). |
| **Default Gateway** | "Cửa chính" để đi ra ngoài mạng nội bộ. | Dòng `default` trong `netstat -rn`. |
| **Interface** | Cánh cửa vật lý/ảo (Wi-Fi, LAN, VPN). | `en0`, `en7`, `lo0`, `utun0`. |
| **NAT** | Router "thay tên đổi họ" để ra Internet. | Khi thấy IP Local khác hoàn toàn IP Public. |
| **TCP/UDP** | Cách thức "vận chuyển" hàng hóa. | TCP (Bảo đảm), UDP (Nhanh/Mất mát). |
| **DNS** | Danh bạ điện thoại (Tên ➔ IP). | Lệnh `dig` hoặc `nslookup`. |

## Ghi chú thêm
- **Private IP**: Địa chỉ được cấp phát bên trong mạng nội bộ (LAN). Các thiết bị khác trên Internet không thể nhìn thấy địa chỉ này.
- **Public IP**: Địa chỉ duy nhất trên toàn cầu, được nhà cung cấp dịch vụ Internet (ISP) cấp. Khi bạn truy cập một website, máy chủ của website đó sẽ nhìn thấy Public IP của bạn (sau khi đã đi qua bước NAT tại Router).
