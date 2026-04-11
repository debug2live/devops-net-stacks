# KIỂM TRA ĐƯỜNG RA CỦA GÓI TIN (ROUTING TABLE)

## Case 3: Kiểm tra "Đường ra" (Routing)
- **Lệnh:** `netstat -rn`
- **Ví dụ:**
  - Nếu dòng `default` trỏ vào `utun0`, bạn đang đi qua mạng VPN.
  - Nếu trỏ vào `en7` hoặc `en0`, bạn đang đi mạng dây trực tiếp hoặc qua Wi-Fi từ Router của nhà.

Bảng routing (Routing Table) giúp máy tính của bạn biết cần gửi gói tin (packet) tới đâu. Mặc định, nếu không tìm được đường dẫn cụ thể nào khác trong mạng nội bộ, gói tin sẽ được đưa vào "Default Gateway" để Router xử lý bước tiếp theo.
