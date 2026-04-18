# 2. Quy trình thiết lập (Setup Steps)

## Bước 1: Tạo Namespace & Bridge
```bash
sudo ip netns add red
sudo ip link add br0 type bridge
sudo ip link set br0 up
```

## Bước 2: Kết nối dây (veth)
```bash
sudo ip link add v-red type veth peer name v-red-br
sudo ip link set v-red netns red
sudo ip link set v-red-br master br0
sudo ip link set v-red-br up
```

## Bước 3: Cấu hình IP & Routing
```bash
sudo ip netns exec red ip addr add 10.0.0.1/24 dev v-red
sudo ip netns exec red ip link set v-red up
sudo ip netns exec red ip link set lo up
sudo ip addr add 10.0.0.254/24 dev br0  # Cấp IP cho Bridge để Host tham gia mạng
sudo ip netns exec red ip route add default via 10.0.0.254
```
