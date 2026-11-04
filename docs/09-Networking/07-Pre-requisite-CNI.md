# Kiến thức nền tảng: CNI

  - Đi tới [Lecture](https://kodekloud.com/topic/prerequsite-cni/)

Trong phần này, chúng ta sẽ tìm hiểu **kiến thức nền tảng về Container Network Interface (CNI)**


![net-7](../../images/net7.PNG)

> 💡 **Bổ sung:** CNI (Container Network Interface) là một bộ tiêu chuẩn (specification) quy định cách một chương trình (CNI plugin) được gọi để cấu hình mạng cho container, cùng với các thư viện hỗ trợ. Thay vì mỗi container runtime (Docker, rkt, containerd, CRI-O...) tự viết lại toàn bộ các bước tạo namespace → tạo veth → gắn vào bridge → cấp IP → bật NAT như các phần trước, CNI chuẩn hóa việc này:
> - **Container runtime** chịu trách nhiệm tạo network namespace, xác định network mà container cần gắn vào, gọi plugin với lệnh `ADD` khi container được tạo và `DEL` khi container bị xóa, truyền cấu hình mạng dưới dạng JSON.
> - **CNI plugin** là một file thực thi, phải hỗ trợ các lệnh `ADD`/`DEL`/`CHECK` (và `VERSION`), chịu trách nhiệm gán IP cho Pod/container, cấu hình route và trả kết quả theo định dạng chuẩn.
>
> Lưu ý: Docker **không** dùng CNI mà dùng chuẩn riêng là CNM (Container Network Model). Vì vậy Kubernetes trước đây phải tạo container Docker với network `none` rồi tự gọi CNI plugin. Từ Kubernetes v1.24 dockershim đã bị gỡ bỏ; hiện nay các runtime theo chuẩn CRI như containerd và CRI-O gọi CNI plugin trực tiếp.

## Các nhà cung cấp Network Plugin bên thứ ba

- [Weave](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-installation)
- [Calico](https://docs.projectcalico.org/getting-started/kubernetes/quickstart)
- [Flannel](https://github.com/coreos/flannel/blob/master/Documentation/kubernetes.md)
- [Cilium](https://github.com/cilium/cilium)

> 💡 **Bổ sung (cập nhật 2025+):**
> - **Weave Net** đã ngừng phát triển: công ty Weaveworks đóng cửa đầu năm 2024 và dự án không còn được bảo trì, nên link tài liệu ở trên có thể không còn truy cập được. Weave vẫn xuất hiện trong bài giảng/lab nhưng không nên dùng cho cluster mới.
> - **Calico** (Tigera): rất phổ biến, hỗ trợ đầy đủ Network Policy, có thể định tuyến bằng BGP hoặc overlay IPIP/VXLAN, có cả eBPF dataplane. Tài liệu hiện nằm tại `docs.tigera.io`.
> - **Cilium**: dựa trên eBPF, là dự án graduated của CNCF, hỗ trợ Network Policy (kể cả L7), có thể thay thế kube-proxy, tích hợp observability (Hubble). Là CNI mặc định của nhiều dịch vụ managed Kubernetes (ví dụ GKE Dataplane V2, tùy chọn trên AKS).
> - **Flannel**: đơn giản, nhẹ, overlay VXLAN, nhưng **không** tự thực thi Network Policy (thường được kết hợp với Calico thành "Canal"). Repo hiện ở `github.com/flannel-io/flannel`.
>
> Khi thi CKA, đề có thể yêu cầu cài một CNI (ví dụ Calico hoặc Flannel) theo link tài liệu được cung cấp; lưu ý chọn CNI hỗ trợ Network Policy nếu đề yêu cầu.

## Xem các CNI Network Plugin

- CNI đi kèm một bộ các network plugin được hỗ trợ sẵn (thường nằm trong `/opt/cni/bin`).

```
$ ls /opt/cni/bin/
bridge  dhcp  flannel  host-device  host-local  ipvlan  loopback  macvlan  portmap  ptp  sample  tuning  vlan
```

> 💡 **Bổ sung:** Các plugin này chia thành nhiều nhóm: plugin tạo interface (`bridge`, `macvlan`, `ipvlan`, `ptp`, `host-device`, `vlan`...), plugin IPAM cấp phát IP (`host-local`, `dhcp`, `static`) và plugin meta (`portmap`, `tuning`, `bandwidth`...). File cấu hình network của CNI nằm trong `/etc/cni/net.d/`.

#### Tài liệu tham khảo

- https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/
