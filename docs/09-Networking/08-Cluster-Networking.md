# Kiến thức nền tảng: Cluster Networking

  - Đi tới [Lecture](https://kodekloud.com/topic/cluster-networking/)

Trong phần này, chúng ta sẽ tìm hiểu **các yêu cầu cơ bản về mạng của cluster**

- Đặt hostname duy nhất cho mỗi node.
- Lấy địa chỉ IP của từng máy (master và worker node). Mỗi node phải có ít nhất một interface nối vào mạng chung, với IP riêng và MAC address duy nhất.
- Kiểm tra các port cần mở.

> 💡 **Bổ sung:** Các port cần mở theo tài liệu kubeadm:
> - **Control plane:** `6443` (kube-apiserver), `2379-2380` (etcd client API và peer), `10250` (kubelet API), `10259` (kube-scheduler), `10257` (kube-controller-manager).
> - **Worker node:** `10250` (kubelet API), `10256` (kube-proxy), `30000-32767` (dải port mặc định của NodePort Service).
>
> Ngoài ra CNI plugin cũng cần port riêng (ví dụ Weave dùng TCP `6783` và UDP `6783/6784`, Flannel VXLAN dùng UDP `8472`, Calico BGP dùng TCP `179`). Lưu ý: các tài liệu cũ ghi port `10251`/`10252` cho scheduler/controller-manager — các port không bảo mật này đã bị loại bỏ, hiện chỉ dùng port HTTPS `10259`/`10257`.

## IP và Hostname

- Xem hostname

```
$ hostname 
```

- Xem địa chỉ IP của máy

```
$ ip a
```


## Đặt hostname

```
$ hostnamectl set-hostname <host-name>

$ exec bash
```

## Xem các port đang lắng nghe trên máy

```
$ netstat -nltp
```

> 💡 **Bổ sung:** `netstat` thuộc gói `net-tools` và có thể không có sẵn trên một số hệ thống mới. Lệnh thay thế tương đương là `ss -nltp` (thuộc gói `iproute2`, luôn có sẵn).

#### Tài liệu tham khảo

- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports
- https://kubernetes.io/docs/concepts/cluster-administration/networking/
