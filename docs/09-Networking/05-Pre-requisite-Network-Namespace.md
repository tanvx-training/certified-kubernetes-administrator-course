# Kiến thức nền tảng: Network Namespaces

  - Đi tới [Lecture](https://kodekloud.com/topic/prerequsite-network-namespaces/)

Trong phần này, chúng ta sẽ tìm hiểu **Network Namespaces**

> 💡 **Bổ sung:** Namespace là cơ chế của Linux kernel để cô lập tài nguyên cho một nhóm process. Có nhiều loại namespace: PID, network, mount, UTS, IPC, user, cgroup. Container (và Pod trong Kubernetes) thực chất là các process chạy trong những namespace riêng. Network namespace cho mỗi container một "ngăn xếp mạng" riêng: interface, bảng định tuyến, bảng ARP, rule iptables riêng — tách biệt hoàn toàn với host.

## Process Namespace

> Bên trong container
```
$ ps aux      
```

> Trên host
```
$ ps aux 

```

> 💡 **Bổ sung:** Bên trong container, `ps aux` chỉ thấy process của chính container đó (process chính có PID 1). Trên host, `ps aux` thấy được cùng process ấy nhưng với một PID khác — vì host nhìn thấy tất cả namespace.

## Network Namespace

```
$ route
```

```
$ arp
```

> 💡 **Bổ sung:** Host có interface, routing table và ARP table riêng. Khi tạo container, ta tạo một network namespace cho nó để container không thấy được các thông tin mạng này của host.

## Tạo Network Namespace

```
$ ip netns add red

$ ip netns add blue
```
- Liệt kê các network namespace

```
$ ip netns
```

## Chạy lệnh bên trong Network Namespace

- Liệt kê các interface trên host

```
$ ip link
```

- Chạy lệnh bên trong network namespace

```
$ ip netns exec red ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

$ ip netns exec blue ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```
- Bạn cũng có thể dùng tùy chọn `-n`. Hai cách cho kết quả như nhau.
```
$ ip -n red link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

> 💡 **Bổ sung:** Trong namespace mới tạo chỉ có interface loopback `lo` (và đang DOWN), không thấy `eth0` hay các interface khác của host.

## ARP và Routing Table

> Trên host
```
$ arp
Address                  HWtype  HWaddress           Flags Mask            Iface
172.17.0.21              ether   02:42:ac:11:00:15   C                     ens3
172.17.0.55              ether   02:42:ac:11:00:37   C                     ens3
```

> Trong network namespace
```
$ ip netns exec red arp
Address                  HWtype  HWaddress           Flags Mask            Iface

$ ip netns exec blue arp
Address                  HWtype  HWaddress           Flags Mask            Iface
```

> Trên host 
```
$ route
```

> Trong network namespace
```
$ ip netns exec red route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface

$ ip netns exec blue route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
```

## Cáp ảo (Virtual Cable)

- Tạo một cáp ảo (cặp veth)
```
$ ip link add veth-red type veth peer name veth-blue
```

> 💡 **Bổ sung:** `veth` (virtual ethernet) luôn được tạo theo cặp, hoạt động như một sợi cáp có hai đầu: gói tin đi vào đầu này sẽ đi ra ở đầu kia. Đây chính là cách CNI plugin nối network namespace của Pod với host.

- Gắn mỗi đầu cáp vào một network namespace
```
$ ip link set veth-red netns red

$ ip link set veth-blue netns blue
```

- Gán địa chỉ IP
```
$ ip -n red addr add 192.168.15.1/24 dev veth-red

$ ip -n blue addr add 192.168.15.2/24 dev veth-blue
```

- Bật (up) các interface trong namespace
```
$ ip -n red link set veth-red up

$ ip -n blue link set veth-blue up
```

- Kiểm tra kết nối
```
$ ip netns exec red ping 192.168.15.2
PING 192.168.15.2 (192.168.15.2) 56(84) bytes of data.
64 bytes from 192.168.15.2: icmp_seq=1 ttl=64 time=0.035 ms
64 bytes from 192.168.15.2: icmp_seq=2 ttl=64 time=0.046 ms

$ ip netns exec red arp
Address                  HWtype  HWaddress           Flags Mask            Iface
192.168.15.2             ether   da:a7:29:c4:5a:45   C                     veth-red

$ ip netns exec blue arp
Address                  HWtype  HWaddress           Flags Mask            Iface
192.168.15.1             ether   92:d1:52:38:c8:bc   C                     veth-blue

```

- Xóa link (xóa một đầu thì đầu kia của cặp veth cũng tự động bị xóa).
```
$ ip -n red link del veth-red
```

> Trên host
```
# Not available
$ arp
Address                  HWtype  HWaddress           Flags Mask            Iface
172.16.0.72              ether   06:fe:61:1a:75:47   C                     ens3
172.17.0.68              ether   02:42:ac:11:00:44   C                     ens3
172.17.0.74              ether   02:42:ac:11:00:4a   C                     ens3
172.17.0.75              ether   02:42:ac:11:00:4b   C                     ens3
```

> 💡 **Bổ sung:** Host không hề biết về các entry ARP của namespace `red`/`blue` — mỗi namespace có bảng ARP riêng.

## Linux Bridge

> 💡 **Bổ sung:** Nối trực tiếp từng cặp namespace bằng veth không mở rộng được khi có nhiều namespace. Giải pháp là tạo một switch ảo (Linux bridge) trên host và nối tất cả namespace vào đó — giống như Docker với `docker0` hay CNI plugin `bridge` với `cni0`.

- Tạo network namespace

```
$ ip netns add red

$ ip netns add blue
``` 
- Để tạo một mạng bridge ảo nội bộ, ta thêm một interface mới kiểu `bridge` trên host
```
$ ip link add v-net-0 type bridge
```
- Interface này hiển thị trên host
```
$ ip link
8: v-net-0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether fa:fd:d4:9b:33:66 brd ff:ff:ff:ff:ff:ff
```
- Hiện nó đang DOWN, hãy bật nó lên
```
$ ip link set dev v-net-0 up
```
- Để nối network namespace vào bridge, ta tạo các cáp ảo
```
$ ip link add veth-red type veth peer name veth-red-br

$ ip link add veth-blue type veth peer name veth-blue-br
```
- Gắn một đầu vào network namespace, đầu còn lại vào bridge
```
$ ip link set veth-red netns red

$ ip link set veth-blue netns blue

$ ip link set veth-red-br master v-net-0

$ ip link set veth-blue-br master v-net-0
```
- Gán địa chỉ IP
```
$ ip -n red addr add 192.168.15.1/24 dev veth-red

$ ip -n blue addr add 192.168.15.2/24 dev veth-blue
```
- Bật các interface trong namespace
```
$ ip -n red link set veth-red up

$ ip -n blue link set veth-blue up
```
- Gán địa chỉ IP cho bridge (để host có thể giao tiếp với các namespace)
```
$ ip addr add 192.168.15.5/24 dev v-net-0
```
- Bật các interface vừa thêm trên host
```
$ ip link set dev veth-red-br up
$ ip link set dev veth-blue-br up
```

> Trên host
```
$ ping 192.168.15.1
```

> Trong namespace
```
$ ip netns exec blue ping 192.168.1.1
Connect: Network is unreachable

$ ip netns exec blue route

$ ip netns exec blue ip route add 192.168.1.0/24 via 192.168.15.5

# Check the IP Address of the host
$ ip a

$ ip netns exec blue ping 192.168.1.1
PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.

$ iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE

$ ip netns exec blue ping 192.168.1.1

$ ip netns exec blue ping 8.8.8.8

$ ip netns exec blue route

$ ip netns exec blue ip route add default via 192.168.15.5

$ ip netns exec blue ping 8.8.8.8
```

> 💡 **Bổ sung:** Giải thích các bước trên:
> - Namespace `blue` không có route tới mạng ngoài `192.168.1.0/24`, nên báo "Network is unreachable". Ta thêm route đi qua bridge `v-net-0` (`192.168.15.5`), lúc này host đóng vai trò gateway (host cần bật `ip_forward`).
> - Gói tin đến được đích nhưng không có phản hồi, vì máy bên ngoài không biết đường quay về mạng riêng `192.168.15.0/24`. Rule `MASQUERADE` (NAT nguồn) thay địa chỉ nguồn bằng IP của host, nhờ vậy phản hồi quay về được.
> - Để ra Internet (`8.8.8.8`), thêm default route qua host.

- Thêm rule port forwarding vào iptables (để bên ngoài truy cập được ứng dụng trong namespace `blue`)

```
$ iptables -t nat -A PREROUTING --dport 80 --to-destination 192.168.15.2:80 -j DNAT
```
```
$ iptables -nvL -t nat
```

> 💡 **Bổ sung:** Lệnh trên là dạng rút gọn trong bài giảng; trên thực tế `--dport` yêu cầu phải chỉ định protocol, tức cần thêm `-p tcp` (ví dụ `iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.15.2:80`). Kube-proxy (ở chế độ iptables) cũng dùng chính các rule DNAT kiểu này để chuyển traffic tới Service về các Pod.
