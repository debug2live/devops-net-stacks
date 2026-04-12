# 4. Nhật ký Sửa lỗi (Troubleshooting Guide)

| Hiện tượng lỗi | Nguyên nhân gốc rễ | Cách kiểm tra | Lệnh sửa lỗi |
| :--- | :--- | :--- | :--- |
| **Network is unreachable** | Thiếu Default Gateway trong Namespace. | `ip netns exec red ip route` | `ip route add default via 10.0.0.254` |
| **Ping Timeout (tại Bridge)** | Host chưa bật chuyển tiếp gói tin. | `cat /proc/sys/net/ipv4/ip_forward` | `sysctl -w net.ipv4.ip_forward=1` |
| **Ping Timeout (tại eth0)** | Thiếu NAT (Masquerade). | `iptables -t nat -L` | `iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE` |
| **DNS failure** | Namespace chưa có cấu hình DNS. | `cat /etc/netns/red/resolv.conf` | Tạo file và thêm `nameserver 8.8.8.8` |

# 5. Kết luận & Mở rộng (Key Takeaways)

- Veth Pair là sợi dây, Bridge là cái Switch, Host IP là Gateway.
- Mọi Container Network (Docker, K8s) đều dựa trên nguyên lý này.
- Luôn dùng `tcpdump` để xác định chính xác gói tin bị "rơi" ở phân đoạn nào.
