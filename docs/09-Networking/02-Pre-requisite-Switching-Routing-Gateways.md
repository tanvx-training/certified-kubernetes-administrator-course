# Kiến thức nền tảng: Switching, Routing và Gateways

  - Đi tới [Lecture](https://kodekloud.com/topic/pre-requisite-switching-routing-gateways-cni-in-kubernetes/)

Trong phần này, chúng ta sẽ tìm hiểu **Switching, Routing và Gateways**

## Switching

- Xem các interface (giao diện mạng) trên host

```
$ ip link
```
- Xem địa chỉ IP gán cho các interface.

```
$ ip addr
```

![net-14](../../images/net14.PNG)

> 💡 **Bổ sung:** Switch kết nối các máy trong **cùng một mạng** (cùng subnet). Hai máy cùng nối vào một switch chỉ có thể giao tiếp với nhau khi mỗi máy có IP thuộc cùng subnet, gán bằng lệnh như `ip addr add 192.168.1.10/24 dev eth0`. Lưu ý: các thay đổi bằng lệnh `ip` chỉ tồn tại tới khi khởi động lại; muốn lưu lâu dài phải cấu hình trong file network của distro (vd `/etc/netplan/*.yaml` trên Ubuntu, `/etc/network/interfaces`, hoặc NetworkManager).

## Routing

- Xem bảng định tuyến (routing table) hiện có trên host.

```
$ route
```
```
$ ip route show
or
$ ip route list
```

- Thêm entry vào bảng định tuyến.

```
$ ip route add 192.168.1.0/24 via 192.168.2.1
```

![net-15](../../images/net15.PNG)

> 💡 **Bổ sung:** Router kết nối **các mạng khác nhau**. Lệnh trên có nghĩa: "để tới mạng `192.168.1.0/24`, hãy gửi gói tin qua gateway `192.168.2.1`" (gateway phải nằm trong mạng mà host đang kết nối trực tiếp). Lệnh `route` thuộc gói `net-tools` đã cũ và nhiều distro không cài sẵn; nên ưu tiên dùng `ip route`.

## Gateways

- Thêm default route (tuyến mặc định).
```
$ ip route add default via 192.168.2.1
```

> 💡 **Bổ sung:** Default route (`default` hay `0.0.0.0/0`) được dùng cho mọi đích không khớp với route cụ thể nào khác — thường là đường ra Internet. Khi có nhiều route khớp, Linux chọn route có prefix dài nhất (cụ thể nhất).

- Kiểm tra IP forwarding đã được bật trên host chưa.
```
$ cat /proc/sys/net/ipv4/ip_forward
0

$ echo 1 > /proc/sys/net/ipv4/ip_forward
```

> 💡 **Bổ sung:** Mặc định Linux **không** chuyển tiếp gói tin từ interface này sang interface khác (`ip_forward = 0`), kể cả khi host có nhiều interface. Muốn một host Linux hoạt động như router thì phải bật IP forwarding. Đây cũng là yêu cầu bắt buộc trên các node Kubernetes (xem phần cài đặt kubeadm: `net.ipv4.ip_forward = 1`). Ghi trực tiếp vào `/proc` chỉ có hiệu lực tới khi reboot.

- Bật chuyển tiếp gói tin (packet forwarding) cho IPv4 một cách lâu dài.
```
$ cat /etc/sysctl.conf

# Uncomment the line
net.ipv4.ip_forward=1
```

- Xem các biến sysctl.
```
$ sysctl -a 
```

- Nạp lại cấu hình sysctl.
```
$ sysctl --system
```

> 💡 **Bổ sung:** Cách được khuyến nghị (và được dùng trong tài liệu cài đặt kubeadm) là tạo một file riêng trong `/etc/sysctl.d/`, ví dụ `/etc/sysctl.d/k8s.conf` chứa dòng `net.ipv4.ip_forward = 1`, sau đó chạy `sudo sysctl --system` để áp dụng. Kiểm tra lại bằng `sysctl net.ipv4.ip_forward`.
