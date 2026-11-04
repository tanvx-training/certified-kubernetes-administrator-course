# Giới thiệu - Kubeadm

Lưu ý rằng phần này chủ yếu để bạn tự thực hành. Theo những gì chúng tôi biết, trước đây chưa từng có câu hỏi thi nào yêu cầu bạn cài đặt cluster. Tuy nhiên, các câu hỏi yêu cầu _upgrade một cluster có sẵn_ thì khá phổ biến!

> 💡 **Bổ sung:** Từ curriculum CKA cập nhật năm 2025, domain "Cluster Architecture, Installation & Configuration" có các mục "Prepare underlying infrastructure for installing a Kubernetes cluster" và "Create and manage Kubernetes clusters using kubeadm". Vì vậy, bạn nên thành thạo các thao tác như cài container runtime và các gói kubeadm/kubelet/kubectl, chạy `kubeadm init`, join node bằng `kubeadm join`, và cài CNI plugin.

  - Xem [Giới thiệu](https://kodekloud.com/topic/introduction-to-deployment-with-kubeadm/)

Nếu bạn dùng máy Apple M1 hoặc M2 (Apple Silicon), hãy làm theo hướng dẫn riêng tại [đây](../../apple-silicon/README.md).

> 💡 **Bổ sung:** Các bước tổng quan để dựng cluster bằng kubeadm:
>
> 1. Chuẩn bị nhiều VM/máy chủ (vd 1 master node và 2 worker node), đảm bảo các máy kết nối được với nhau qua network, có hostname, địa chỉ MAC và `product_uuid` khác nhau, và **tắt swap** (hoặc cấu hình kubelet để chấp nhận swap).
> 2. Trên **mọi node**: nạp kernel module `overlay` và `br_netfilter`, bật các tham số sysctl `net.bridge.bridge-nf-call-iptables` và `net.ipv4.ip_forward`.
> 3. Trên **mọi node**: cài container runtime (containerd hoặc CRI-O) và cấu hình cgroup driver `systemd` cho cả runtime lẫn kubelet.
> 4. Trên **mọi node**: cài `kubeadm`, `kubelet`, `kubectl` từ repository `pkgs.k8s.io`, sau đó chạy `apt-mark hold` để tránh bị tự động upgrade.
> 5. Trên **master node**: chạy `kubeadm init`. Lệnh này thực hiện các pha (phase) chính:
>    - `preflight`: kiểm tra điều kiện của hệ thống.
>    - `certs`: tạo CA và các certificate trong `/etc/kubernetes/pki`.
>    - `kubeconfig`: tạo các file `admin.conf`, `kubelet.conf`, `controller-manager.conf`, `scheduler.conf`.
>    - `control-plane` và `etcd`: tạo manifest static Pod cho các thành phần control plane trong `/etc/kubernetes/manifests`.
>    - `upload-config`, `mark-control-plane`, `bootstrap-token`.
>    - `addon`: cài CoreDNS và kube-proxy.
> 6. Thiết lập kubeconfig cho người dùng bằng cách copy `/etc/kubernetes/admin.conf` vào `~/.kube/config`.
> 7. Cài một CNI plugin (Flannel, Calico, Cilium, Weave...). Trước khi có CNI, các node ở trạng thái `NotReady` và CoreDNS ở trạng thái `Pending`.
> 8. Trên **worker node**: chạy lệnh `kubeadm join` mà `kubeadm init` in ra. Nếu làm mất lệnh này hoặc token đã hết hạn (mặc định sau 24 giờ), có thể tạo lại bằng `kubeadm token create --print-join-command`.
