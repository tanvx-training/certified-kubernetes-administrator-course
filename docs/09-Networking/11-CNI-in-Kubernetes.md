# CNI trong Kubernetes

  - Đi tới [Lecture](https://kodekloud.com/topic/cni-in-kubernetes/)

Trong phần này, chúng ta sẽ tìm hiểu **Container Networking Interface (CNI) trong Kubernetes**

## Cấu hình CNI

![net-1](../../images/net1.PNG)

> ⚠️ **Lưu ý phiên bản:** Hình trên (từ bài giảng gốc) cho thấy kubelet được cấu hình với các tùy chọn `--network-plugin=cni`, `--cni-bin-dir`, `--cni-conf-dir`. Các flag này **đã bị loại bỏ từ Kubernetes v1.24** cùng với dockershim. Hiện nay việc gọi CNI plugin là trách nhiệm của **container runtime** (containerd hoặc CRI-O), không phải kubelet. Với containerd, đường dẫn CNI được cấu hình trong `/etc/containerd/config.toml` (mục plugin CRI, các khóa `bin_dir` và `conf_dir`, mặc định là `/opt/cni/bin` và `/etc/cni/net.d`).

- Kiểm tra trạng thái service kubelet

```
$ systemctl status kubelet.service
```

## Xem các tùy chọn của kubelet

```
$ ps -aux | grep kubelet
```

> 💡 **Bổ sung:** Trên cluster dựng bằng kubeadm, phần lớn cấu hình kubelet nằm trong file `/var/lib/kubelet/config.yaml`, còn các flag dòng lệnh nằm trong `/var/lib/kubelet/kubeadm-flags.env` (ví dụ `--container-runtime-endpoint=unix:///var/run/containerd/containerd.sock`).

## Kiểm tra các plugin được hỗ trợ

- Xem tất cả plugin được hỗ trợ có trong thư mục `/opt/cni/bin`.

```
$ ls /opt/cni/bin

```

## Kiểm tra CNI plugin đang dùng

- Xem cấu hình CNI plugin sẽ được sử dụng.

```
ls /etc/cni/net.d

```

> 💡 **Bổ sung:** Nếu thư mục `/etc/cni/net.d` có nhiều file cấu hình, runtime sẽ chọn file đầu tiên theo thứ tự alphabet (vì vậy các file thường có tiền tố số như `10-flannel.conflist`, `10-calico.conflist`). Nếu chưa có CNI nào được cài, các node sẽ ở trạng thái `NotReady` và Pod CoreDNS sẽ ở `Pending`/`ContainerCreating` — đây là dấu hiệu quen thuộc khi troubleshooting cluster mới.

## Định dạng file cấu hình

![net-2](../../images/net2.PNG)

> 💡 **Bổ sung:** Các trường chính trong file cấu hình CNI:
> - `cniVersion`, `name`: phiên bản spec CNI và tên network.
> - `type`: tên plugin (file thực thi trong `/opt/cni/bin`), ví dụ `bridge`.
> - `bridge`: tên bridge sẽ tạo trên node (ví dụ `cni0`).
> - `isGateway`: có gán IP cho bridge để nó làm gateway cho Pod hay không.
> - `ipMasq`: có thêm rule NAT (masquerade) cho traffic đi ra ngoài hay không.
> - `ipam`: cấu hình cấp phát IP — `type: host-local` nghĩa là IP được quản lý cục bộ trên node, `subnet` là dải IP cấp cho Pod, `routes` là các route sẽ thêm vào Pod.
>
> File dạng `.conflist` chứa danh sách `plugins` được gọi theo chuỗi (ví dụ `bridge` rồi `portmap`).

#### Tài liệu tham khảo

- https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/
- https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/
