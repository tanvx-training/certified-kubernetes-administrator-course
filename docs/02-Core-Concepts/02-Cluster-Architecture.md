# Kiến trúc cluster (Cluster Architecture)

  - Xem [Video hướng dẫn](https://kodekloud.com/topic/cluster-architecture/)

Trong phần này, chúng ta sẽ tìm hiểu kiến trúc Kubernetes ở mức tổng quan.
- Cái nhìn toàn cảnh (từ độ cao 10.000 feet) về kiến trúc Kubernetes

  ![Kubernetes Architecture](../../images/k8s-arch.PNG)
  
  ![Kubernetes Architecture 1](../../images/k8s-arch1.PNG)

> 💡 **Bổ sung — tóm tắt các thành phần:**
> - **Control plane** (trước đây gọi là master node — tài liệu mới dùng thuật ngữ "control plane node"): quản lý toàn bộ cluster, gồm:
>   - `kube-apiserver`: "cửa ngõ" duy nhất của cluster; mọi thành phần khác và `kubectl` đều giao tiếp qua nó.
>   - `etcd`: kho key-value lưu toàn bộ trạng thái của cluster.
>   - `kube-scheduler`: quyết định Pod sẽ chạy trên node nào.
>   - `kube-controller-manager`: chạy các controller (Node, ReplicaSet, Deployment...) để đưa trạng thái thực tế về trạng thái mong muốn.
>   - `cloud-controller-manager` (tùy chọn): tích hợp với nhà cung cấp cloud (load balancer, route, node...).
> - **Worker node**: nơi chạy workload, gồm:
>   - `kubelet`: agent trên mỗi node, nhận chỉ thị từ kube-apiserver và quản lý Pod/container thông qua container runtime.
>   - `kube-proxy`: thiết lập rule mạng (iptables/IPVS/nftables) để hiện thực Service.
>   - Container runtime tương thích CRI (containerd, CRI-O...).
> - Trong cluster dựng bằng kubeadm, các thành phần control plane chạy dưới dạng **static Pod** trong namespace `kube-system` (manifest nằm ở `/etc/kubernetes/manifests/`), còn kubelet chạy dưới dạng service systemd trên mọi node.

Tài liệu tham khảo K8s:
- https://kubernetes.io/docs/concepts/architecture/
