# 3. Các bước kiểm chứng (Testing & Verification)

Để đảm bảo gói tin đi đúng lộ trình, ta thực hiện các bài test theo thứ tự từ gần đến xa:

## Test 1: Red có thấy "Cửa ngõ" (Bridge) không?
Từ phòng Red, ping tới IP của Bridge.
```bash
sudo ip netns exec red ping 10.0.0.254
```
**Thành công:** Red và Host đã thông Layer 2 và Layer 3 nội bộ.

## Test 2: Host có nhận được gói tin từ Red không? (Bắt mạch tại Bridge)
Mở một Terminal khác chạy tcpdump trên máy Host:
```bash
sudo tcpdump -i br0 icmp
```
**Dấu hiệu:** Nếu thấy dòng `10.0.0.1 > 10.0.0.254`, nghĩa là Bridge đã nhận được dữ liệu.

## Test 3: Gói tin có đi ra card mạng thật không?
```bash
sudo tcpdump -i enp0s1 icmp
```
**Dấu hiệu:** Nếu thấy `10.0.0.1 > 8.8.8.8`, nghĩa là `ip_forward` đã hoạt động.
