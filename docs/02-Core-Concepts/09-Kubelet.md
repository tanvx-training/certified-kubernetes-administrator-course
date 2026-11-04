# Kubelet
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/kubelet/)
  
Trong phần này, chúng ta sẽ tìm hiểu kubelet.

#### Kubelet là đầu mối liên lạc duy nhất (của mỗi node) với cluster Kubernetes
- **`kubelet`** sẽ tạo Pod trên các node, còn scheduler chỉ quyết định Pod nào chạy ở đâu.

  ![kubelet](../../images/kubelet.PNG)

> 💡 **Bổ sung:** Nhiệm vụ chính của kubelet: đăng ký node với cluster; nhận yêu cầu chạy Pod từ kube-apiserver và gọi container runtime (qua CRI) để pull image, chạy container; liên tục giám sát Pod/container (bao gồm chạy liveness/readiness probe) và báo cáo trạng thái node, Pod về kube-apiserver. kubelet cũng là thành phần tạo ra các **static Pod** từ thư mục manifest (`staticPodPath`).
  
## Cài đặt kubelet
- Kubeadm không tự triển khai kubelet. Bạn phải tự tải về và cài đặt kubelet thủ công.
- Tải binary kubelet từ trang phát hành của Kubernetes [kubelet](https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kubelet). Ví dụ: để tải kubelet v1.13.0, chạy lệnh dưới đây.
  ```
  $ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kubelet
  ```
- Giải nén
- Chạy nó dưới dạng service

  ![kubelet1](../../images/kubelet1.PNG)

> 💡 **Bổ sung:** Trên thực tế, với cluster kubeadm, kubelet thường được cài từ package của hệ điều hành (cùng với `kubeadm`, `kubectl`) trên **mọi** node, kể cả control plane node, và chạy dưới dạng service systemd. Vì kubelet không phải là Pod, bạn sẽ không thấy nó trong `kubectl get pods -n kube-system`. Các lệnh hay dùng khi troubleshoot node `NotReady`:
> - `systemctl status kubelet`, `systemctl restart kubelet`
> - `journalctl -u kubelet` để xem log
> - File cấu hình kubelet (kubeadm): `/var/lib/kubelet/config.yaml`; kubeconfig của kubelet: `/etc/kubernetes/kubelet.conf`
  
## Xem các tùy chọn của kubelet
- Bạn cũng có thể xem tiến trình đang chạy và các tùy chọn đang có hiệu lực bằng cách liệt kê tiến trình trên worker node và tìm kubelet.
  ``` 
  $ ps -aux |grep kubelet
  ```
  
  ![kubelet2](../../images/kubelet2.PNG)

Tài liệu tham khảo K8s:
- https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/
- https://kubernetes.io/docs/concepts/overview/components/
- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/kubelet-integration/
