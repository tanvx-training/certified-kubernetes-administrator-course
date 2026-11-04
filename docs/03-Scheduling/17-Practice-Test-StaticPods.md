# Practice Test - Static Pods (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-static-pods/)
  
Lời giải cho bài practice test - Static Pods
- Chạy lệnh kubectl get pods --all-namespaces và tìm những Pod có hậu tố -controlplane trong tên (đó là các static Pod trên node controlplane)
  
  <details>

  ```
  $ kubectl get pods --all-namespaces
  ```
  </details>

- Thành phần nào dưới đây KHÔNG được triển khai dưới dạng static Pod? (Gợi ý: `coredns` là Deployment, `kube-proxy` là DaemonSet; còn `kube-apiserver`, `kube-controller-manager`, `kube-scheduler`, `etcd` là static Pod.)

  <details>

  ```
  $ kubectl get pods --all-namespaces
  ```
  </details>

- Thành phần nào dưới đây KHÔNG được triển khai dưới dạng static Pod?

  <details>

  ```
  $ kubectl get pods --all-namespaces
  ```
  </details>

- Chạy lệnh kubectl get pods --all-namespaces -o wide (xem các static Pod nằm trên node nào)

  <details>

  ```
  $ kubectl get pods --all-namespaces -o wide
  ```
  </details>

- Chạy lệnh ps -aux | grep kubelet và xác định file config - --config=/var/lib/kubelet/config.yaml. Sau đó tìm trường staticPodPath trong file config này.

  <details>

  ```
  $ ps -aux | grep kubelet
  ```
  </details>

- Đếm số file trong thư mục /etc/kubernetes/manifests

- Kiểm tra image được khai báo trong file manifest /etc/kubernetes/manifests/kube-apiserver.yaml.

- Tạo file định nghĩa Pod trong thư mục manifests. Dùng lệnh kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml (lưu ý: `--dry-run` không kèm giá trị đã deprecated, phải dùng `--dry-run=client`)
  
  <details>

  ```
  $ kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
  ```
  </details>

- Chỉ cần sửa file định nghĩa static Pod và lưu lại (kubelet sẽ tự tạo lại Pod với image mới)

  <details>

  ```
  /etc/kubernetes/manifests/static-busybox.yaml
  ```
  </details>

  HOẶC
  
  <details>

  Chạy lại lệnh với image tag mới (đã sửa lỗi thiếu dấu cách trước `--dry-run=client` ở bản gốc):

  ```
  Run the command with updated image tag:
  kubectl run --restart=Never --image=busybox:1.28.4 static-busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
  ```
  </details>

- Xác định static Pod được tạo trên node nào, SSH vào node đó và xóa file định nghĩa Pod. Nếu không biết IP của node, chạy lệnh kubectl get nodes -o wide để lấy IP, rồi SSH vào node bằng IP đó (hoặc `ssh node01`). Để tìm đường dẫn manifest của static Pod, xem file /var/lib/kubelet/config.yaml trên node01.

  <details>

  ```
  $ kubectl get pods -o wide
  $ kubectl get nodes -o wide
  $ ssh <ip address of pod>
  $ grep staticPodPath /var/lib/kubelet/config.yaml
  $ node01 $ rm -rf /etc/just-to-mess-with-you/greenbox.yaml
  ```

  Lưu ý: `<ip address of pod>` ở trên thực chất là IP của **node** (không phải Pod). Sau khi xóa file, quay lại controlplane và chạy `kubectl get pods` để xác nhận Pod `static-greenbox` đã biến mất. Nếu chỉ dùng `kubectl delete pod`, Pod sẽ được kubelet tạo lại ngay vì file manifest vẫn còn.
  </details>
