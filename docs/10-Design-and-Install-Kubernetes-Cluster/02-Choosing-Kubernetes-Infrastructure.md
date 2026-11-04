# Lựa chọn hạ tầng cho Kubernetes

  - Xem [Bài giảng](https://kodekloud.com/topic/choosing-kubernetes-infrastructure/)

> 💡 **Bổ sung:** Các lựa chọn phổ biến để chạy Kubernetes:
>
> - **Máy local (học tập):**
>   - Kubernetes chạy native trên Linux; trên Windows/macOS cần VM (VirtualBox, Hyper-V, Multipass...) hoặc container.
>   - *minikube:* tạo cluster single-node trong VM hoặc container.
>   - *kind* (Kubernetes IN Docker): chạy mỗi node dưới dạng một container, có thể tạo cluster nhiều node.
>   - *kubeadm:* công cụ chính thức để bootstrap cluster single-node hoặc multi-node trên các VM/máy chủ đã chuẩn bị sẵn. Đây là công cụ được dùng trong kỳ thi CKA.
> - **Turnkey solution:** bạn tự cấp phát VM, sau đó dùng công cụ/script để triển khai cluster (vd kubeadm, Kubespray, OpenShift, Rancher RKE2...). Bạn tự chịu trách nhiệm vận hành và upgrade VM lẫn cluster.
> - **Hosted/managed solution (Kubernetes-as-a-Service):** nhà cung cấp cloud quản lý control plane, bạn chỉ quản lý worker node hoặc node group/pool, ví dụ Google Kubernetes Engine (GKE), Amazon EKS, Azure AKS. Thường bạn không truy cập trực tiếp được vào master node.
>
> Lưu ý: dockershim đã bị gỡ khỏi kubelet từ v1.24, nên container runtime hiện nay thường là **containerd** hoặc **CRI-O** (cả hai đều tuân theo CRI).
