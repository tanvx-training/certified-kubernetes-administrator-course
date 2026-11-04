# Pod Networking

  - Đi tới [Lecture](https://kodekloud.com/topic/pod-networking/)

Trong phần này, chúng ta sẽ tìm hiểu **Pod Networking (mạng của Pod)**

> 💡 **Bổ sung:** Kubernetes không tự triển khai mạng cho Pod mà chỉ đặt ra **mô hình mạng (networking model)** yêu cầu giải pháp mạng phải đáp ứng:
> - Mỗi Pod phải có một địa chỉ IP riêng.
> - Mọi Pod trên cùng một node phải giao tiếp được với nhau.
> - Mọi Pod phải giao tiếp được với Pod ở node khác **mà không cần NAT**.
>
> Phần dưới đây mô phỏng cách tự xây dựng mạng như vậy bằng tay, dựa trên các kiến thức network namespace, bridge, veth và routing ở các phần trước. Trên thực tế, CNI plugin (Calico, Cilium, Flannel...) sẽ làm tự động những việc này.

- Thêm một mạng bridge trên mỗi node

> node01
```
$ ip link add v-net-0 type bridge
```
> node02
```
$ ip link add v-net-0 type bridge
```

> node03
```
$ ip link add v-net-0 type bridge
```

- Hiện bridge đang DOWN, hãy bật nó lên.

> node01
```
$ ip link set dev v-net-0 up
```

> node02
```
$ ip link set dev v-net-0 up
```

> node03
```
$ ip link set dev v-net-0 up
```

- Đặt địa chỉ IP cho interface bridge (mỗi node dùng một subnet riêng)

> node01
```
$ ip addr add 10.244.1.1/24 dev v-net-0
```

> node02
```
$ ip addr add 10.244.2.1/24 dev v-net-0
```

> node03
```
$ ip addr add 10.244.3.1/24 dev v-net-0
```

![net-11](../../images/net11.PNG)

> 💡 **Bổ sung:** Với mỗi container (Pod) được tạo, cần thực hiện các bước giống hệt phần Network Namespace: tạo cặp veth, gắn một đầu vào namespace của container và đầu kia vào bridge, gán IP trong subnet của node, thêm default route qua IP của bridge, rồi bật interface. Bài giảng gom các bước này vào một script (ví dụ `net-script.sh`) để chạy cho mỗi container — đây chính là ý tưởng của CNI plugin. Lúc này các Pod trong cùng node đã nói chuyện được với nhau.

- Kiểm tra kết nối tới Pod ở node khác

```
$ ping 10.244.2.2
Connect: Network is unreachable
```

- Thêm route vào routing table
```
$ ip route add 10.244.2.2 via 192.168.1.12
```

> node01
```
$ ip route add 10.244.2.2 via 192.168.1.12

$ ip route add 10.244.3.2 via 192.168.1.13
```

> node02
```
$ ip route add 10.244.1.2 via 192.168.1.11

$ ip route add 10.244.3.2 via 192.168.1.13

```

> node03
```
$ ip route add 10.244.1.2 via 192.168.1.11

$ ip route add 10.244.2.2 via 192.168.1.12
```

> 💡 **Bổ sung:** Thực tế nên route theo cả subnet của node (ví dụ `ip route add 10.244.2.0/24 via 192.168.1.12`) thay vì từng IP Pod. Tuy vậy, việc cấu hình route thủ công trên mọi node vẫn không mở rộng được khi cluster lớn.

- Gom lại thành một mạng lớn duy nhất (ví dụ `10.244.0.0/16`), cấu hình route tập trung trên router làm default gateway

![net-12](../../images/net12.PNG)


## Container Network Interface

![net-13](../../images/net13.PNG)

> 💡 **Bổ sung:** Để script trên chạy tự động mỗi khi Pod được tạo, nó phải tuân theo chuẩn CNI: có phần `ADD` (khi container được tạo) và `DEL` (khi container bị xóa). Khi tạo Pod, container runtime (containerd/CRI-O) đọc cấu hình CNI trong `/etc/cni/net.d/`, tìm file thực thi plugin trong `/opt/cni/bin/` và chạy plugin với lệnh `ADD`, truyền vào ID container và network namespace.
>
> Để kiểm tra dải IP Pod của cluster, có thể xem `kubectl get nodes -o jsonpath='{.items[*].spec.podCIDR}'` (khi controller-manager cấp podCIDR cho node) hoặc cấu hình của CNI plugin.

#### Tài liệu tham khảo

- https://kubernetes.io/docs/concepts/workloads/pods/
- https://kubernetes.io/docs/concepts/services-networking/
