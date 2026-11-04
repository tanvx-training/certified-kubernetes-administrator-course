# IPAM weave

  - Xem [Bài giảng](https://kodekloud.com/topic/ipam-weave/)

- Quản lý địa chỉ IP (IP Address Management - IPAM) trong Kubernetes cluster

![net-3](../../images/net3.PNG)


- Cách Weaveworks quản lý địa chỉ IP trong Kubernetes cluster

![net-4](../../images/net4.PNG)

> 💡 **Bổ sung:**
> - Theo đặc tả CNI, việc cấp phát IP cho Pod là trách nhiệm của plugin CNI. Plugin CNI thường ủy quyền phần này cho một **IPAM plugin** như `host-local` (mỗi node giữ một dải IP riêng, lưu trạng thái cục bộ trên node) hoặc `dhcp`. Cấu hình IPAM nằm trong mục `ipam` của file cấu hình CNI tại `/etc/cni/net.d/`.
> - Weave Net mặc định cấp phát IP cho Pod từ dải `10.32.0.0/12` (từ `10.32.0.1` đến `10.47.255.254`), và tự chia dải này giữa các peer (node). Có thể đổi dải bằng biến môi trường `IPALLOC_RANGE` trong DaemonSet `weave-net`. Dải này không được trùng với mạng của node hay dải IP Service.
> - Để kiểm tra nhanh trên một node: xem `/etc/cni/net.d/` để biết CNI nào đang dùng và cấu hình IPAM của nó; với kubeadm, dải IP Pod của cluster được đặt qua `--pod-network-cidr` (thể hiện trong `kube-controller-manager` qua flag `--cluster-cidr`).


## Tài liệu tham khảo

- https://www.weave.works/docs/net/latest/kubernetes/kube-addon/
- https://kubernetes.io/docs/concepts/cluster-administration/networking/ 
