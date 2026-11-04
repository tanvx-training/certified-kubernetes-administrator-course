# Thiết kế một Kubernetes Cluster

  - Xem [Bài giảng](https://kodekloud.com/topic/design-a-kubernetes-cluster/)

> 💡 **Bổ sung:** Khi thiết kế một cluster, hãy trả lời các câu hỏi sau trước khi chọn công cụ cài đặt:
>
> - **Mục đích sử dụng:**
>   - *Học tập:* một cluster single-node như minikube, kind hoặc một cluster kubeadm nhỏ trên VM local là đủ.
>   - *Phát triển/kiểm thử:* cluster nhiều node, thường 1 master node và vài worker node, có thể dùng kubeadm hoặc dịch vụ managed (GKE, EKS, AKS).
>   - *Production:* control plane có tính sẵn sàng cao (HA, nhiều master node), chạy trên on-premises với kubeadm hoặc trên cloud dạng managed.
> - **Quy mô:** Kubernetes (v1.30+) chính thức hỗ trợ tới tối đa khoảng 5.000 node, 150.000 Pod, 300.000 container và không quá 110 Pod mỗi node. Kích thước VM của control plane cần tăng theo số lượng node.
> - **Hạ tầng:** on-premises hay cloud (AWS, GCP, Azure...), managed hay tự quản lý (self-hosted).
> - **Workload:** loại ứng dụng (web, big data, AI/ML có GPU...), nhu cầu storage (SSD hiệu năng cao, network storage, PersistentVolume động qua CSI), lưu lượng network.
> - **Phân tách vai trò node:** thông thường không chạy workload trên master node. kubeadm tự động thêm taint `node-role.kubernetes.io/control-plane:NoSchedule` lên control plane node để ngăn điều đó.
> - **Tách etcd:** ở cluster lớn, có thể tách etcd ra các node riêng (mô hình external etcd, xem bài [ETCD in HA](04-ETCD-in-HA.md)).
