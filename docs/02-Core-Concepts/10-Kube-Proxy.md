# Kube Proxy
- Xem [Video hướng dẫn](https://kodekloud.com/topic/kube-proxy/)

Trong phần này, chúng ta sẽ tìm hiểu kube-proxy.

Trong một cluster Kubernetes, mọi Pod đều có thể kết nối tới mọi Pod khác. Điều này đạt được bằng cách triển khai một giải pháp mạng cho Pod (pod network) lên cluster.
- Kube-Proxy là một tiến trình chạy trên mỗi node trong cluster Kubernetes.
  
  ![kube-proxy](../../images/kube-proxy.PNG)

> 💡 **Bổ sung:** IP của Pod có thể thay đổi bất cứ lúc nào, nên ứng dụng giao tiếp với nhau qua **Service** (có IP ổn định). Tuy nhiên Service chỉ là một đối tượng ảo — không có tiến trình hay network interface nào thực sự lắng nghe trên IP của Service. Nhiệm vụ của kube-proxy là theo dõi các Service và endpoint mới qua kube-apiserver, rồi tạo các rule chuyển tiếp (forwarding) trên mỗi node để traffic gửi tới IP của Service được chuyển tới IP của các Pod phía sau. kube-proxy hỗ trợ các mode `iptables` (mặc định trên Linux), `ipvs`, và `nftables` (có ở các bản Kubernetes gần đây). Một số plugin CNI (ví dụ Cilium) có thể thay thế hoàn toàn kube-proxy.
  
## Cài đặt kube-proxy - Thủ công
- Tải binary kube-proxy từ trang phát hành của Kubernetes [kube-proxy](https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-proxy). Ví dụ: để tải kube-proxy v1.13.0, chạy lệnh dưới đây.
  ```
  $ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-proxy
  ```
- Giải nén
- Chạy nó dưới dạng service

  ![kube-proxy1](../../images/kube-proxy1.PNG)

## Xem các tùy chọn của kube-proxy - kubeadm
- Nếu bạn dựng cluster bằng công cụ kubeadm, kubeadm sẽ triển khai kube-proxy dưới dạng Pod trong namespace kube-system. Thực tế nó được triển khai dưới dạng một DaemonSet, nên **mỗi node** trong cluster (không chỉ master node) đều có một Pod kube-proxy.
  ```
  $ kubectl get pods -n kube-system
  ```
  ![kube-proxy2](../../images/kube-proxy2.PNG)

> 💡 **Bổ sung:** Bản gốc ghi "triển khai dưới dạng DaemonSet trên master node" — chính xác là DaemonSet chạy trên tất cả các node. Kiểm tra bằng `kubectl get daemonset kube-proxy -n kube-system`. Với kubeadm, cấu hình của kube-proxy (ví dụ `mode`, `clusterCIDR`) nằm trong ConfigMap `kube-proxy` ở namespace `kube-system`.
  
  
Tài liệu tham khảo K8s:
- https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/
- https://kubernetes.io/docs/concepts/overview/components/
