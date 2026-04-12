# Nhật ký Sửa lỗi (Troubleshooting Guide)

Hãy thực hiện kiểm tra theo đúng thứ tự "từ trong ra ngoài" để phát hiện lỗi khi gói tin không thể ra Internet.

## 🛠️ Các bước kiểm tra Workflow (Troubleshooting Guide)

### Trạm 1: Bảng định tuyến nội bộ (Routing Table)
Trước khi gói tin rời đi, nó phải biết "cửa ra" là ai.
- **Kiểm tra:** `ip netns exec red ip route`
- **Điều kiện cần:** Phải có dòng `default via 10.0.0.254`. Nếu thiếu, lỗi sẽ là `Network is unreachable`.
- **Bản chất:** Đây là nơi chỉ định Gateway. Nếu không có "biển chỉ đường" này, gói tin sẽ không biết phải nhảy vào interface nào.

### Trạm 2: Kết nối vật lý ảo (Veth & Bridge)
Gói tin phải di chuyển từ Namespace tới máy Host.
- **Kiểm tra:** `brctl show br0` (hoặc `bridge link show`)
- **Điều kiện cần:** Interface `v-red-br` phải nằm trong danh sách "interfaces" của Bridge.
- **Cách test:** `ping 10.0.0.254` (Ping từ Namespace tới IP của Bridge). Nếu thông, nghĩa là "dây cáp" ảo đã cắm đúng chỗ.

### Trạm 3: Chốt chặn Kernel (IP Forwarding)
Gói tin đã tới máy Host, nhưng Host có chịu "tiếp tay" đẩy nó đi tiếp không?
- **Kiểm tra:** `cat /proc/sys/net/ipv4/ip_forward`
- **Điều kiện cần:** Giá trị phải là `1`.
- **Dấu hiệu lỗi:** Timeout. Bạn dùng `tcpdump -i br0` thấy gói tin tới, nhưng `tcpdump -i eth0` không thấy gói tin đi ra.

### Trạm 4: Hộ chiếu ra khơi (NAT / Masquerade)
Gói tin chuẩn bị rời máy Host để ra Internet. Nó cần một IP "hợp pháp" (Public IP).
- **Kiểm tra:** `sudo iptables -t nat -L -n`
- **Điều kiện cần:** Có quy tắc `MASQUERADE` cho dải `10.0.0.0/24`.
- **Bản chất:** Thay địa chỉ nguồn `10.0.0.1` bằng IP của máy Host. Nếu thiếu bước này, gói tin ra được Internet nhưng Google không biết đường trả về (vì IP 10.x là IP lậu/nội bộ).

### Trạm 5: Danh bạ điện thoại (DNS Resolution)
Gói tin đã đi và về bằng IP thành công, nhưng bạn muốn dùng tên miền (`google.com`).
- **Kiểm tra:** `cat /etc/netns/red/resolv.conf`
- **Điều kiện cần:** Phải có `nameserver 8.8.8.8`.
- **Dấu hiệu lỗi:** `Temporary failure in name resolution`.

---

## 📝 Tóm tắt các bước Debug nhanh (Cheat Sheet)

- **Mất phương hướng?** Check `ip route` (Default Gateway).
- **Đứt dây?** Check `brctl show` & `ping 10.0.0.254`.
- **Bị chặn cửa?** Check `ip_forward=1`.
- **Không có hộ chiếu?** Check `iptables MASQUERADE`.
- **Không biết tên?** Check `resolv.conf`.

---

## Kết luận & Mở rộng (Key Takeaways)

- Veth Pair là sợi dây, Bridge là cái Switch, Host IP là Gateway.
- Mọi Container Network (Docker, K8s) đều dựa trên nguyên lý này.
- Luôn dùng `tcpdump` để xác định chính xác gói tin bị "rơi" ở phân đoạn nào.
