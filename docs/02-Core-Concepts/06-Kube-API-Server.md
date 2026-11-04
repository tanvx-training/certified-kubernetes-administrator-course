# Kube API Server
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/kube-api-server/)
  
Trong phần này, chúng ta sẽ nói về kube-apiserver trong Kubernetes

#### Kube-apiserver là thành phần chính (trung tâm) của Kubernetes.
- Kube-apiserver chịu trách nhiệm **`authenticating`** (xác thực), **`validating`** (kiểm tra hợp lệ) các request, **`retrieving`** (truy xuất) và **`Updating`** (cập nhật) dữ liệu trong kho key-value ETCD. Thực tế, kube-apiserver là thành phần duy nhất tương tác trực tiếp với kho dữ liệu etcd. Các thành phần khác như kube-scheduler, kube-controller-manager và kubelet đều dùng API Server để cập nhật trạng thái cluster trong phạm vi của mình.
  
  ![post](../../images/post.PNG)

> 💡 **Bổ sung — luồng xử lý khi tạo một Pod:**
> 1. Request (từ `kubectl` hoặc gọi API trực tiếp) đi tới kube-apiserver, được **xác thực (authentication)**, **phân quyền (authorization)** và đi qua các **admission controller**, sau đó được kiểm tra hợp lệ.
> 2. kube-apiserver tạo đối tượng Pod (chưa gán node), ghi vào etcd và trả kết quả cho người dùng.
> 3. kube-scheduler liên tục theo dõi (watch) kube-apiserver, phát hiện Pod chưa có node, chọn node phù hợp và báo lại cho kube-apiserver.
> 4. kube-apiserver cập nhật thông tin vào etcd, rồi chuyển thông tin tới kubelet trên worker node tương ứng.
> 5. kubelet tạo Pod trên node và yêu cầu container runtime chạy image; sau đó báo trạng thái lại cho kube-apiserver, kube-apiserver lại ghi vào etcd.
  
## Cài đặt kube-apiserver

- Nếu bạn dựng kube-apiserver bằng công cụ **`kubeadm`** thì không cần quan tâm phần này, nhưng nếu dựng theo cách "the hard way" thì kube-apiserver có sẵn dưới dạng binary trên trang phát hành của Kubernetes.
  - Ví dụ: bạn có thể tải binary kube-apiserver v1.13.0 tại đây [kube-apiserver](https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-apiserver)
    ```
    $ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-apiserver
    ```

> 💡 **Bổ sung:** Bucket `storage.googleapis.com/kubernetes-release` là địa chỉ cũ; hiện nay binary Kubernetes được phân phối qua `dl.k8s.io`, ví dụ `https://dl.k8s.io/release/v1.31.0/bin/linux/amd64/kube-apiserver`.
 
 ![kube-apiserver](../../images/kube-apiserver.PNG)
 
## Xem kube-apiserver - Kubeadm
- kubeadm triển khai kube-apiserver dưới dạng một Pod trong namespace kube-system trên master node.
  ```
  $ kubectl get pods -n kube-system
  ```
   
  ![kube-apiserver1](../../images/kube-apiserver1.PNG)
   
## Xem các tùy chọn của kube-apiserver - Kubeadm
- Bạn có thể xem các tùy chọn trong file định nghĩa Pod nằm tại **`/etc/kubernetes/manifests/kube-apiserver.yaml`**
  ```
  $ cat /etc/kubernetes/manifests/kube-apiserver.yaml
  ```
  
  ![kube-apiserver2](../../images/kube-apiserver2.PNG)

> 💡 **Bổ sung:** Đây là manifest của **static Pod** — kubelet theo dõi thư mục `/etc/kubernetes/manifests/` và tự tạo lại Pod khi file thay đổi. Vì vậy muốn đổi cấu hình kube-apiserver (ví dụ thêm admission plugin qua `--enable-admission-plugins`), bạn chỉ cần sửa file này rồi đợi Pod khởi động lại; **không** sửa bằng `kubectl edit`. Nếu sửa sai khiến kube-apiserver không lên, `kubectl` sẽ không dùng được — khi đó dùng `crictl ps -a` và `crictl logs <container-id>` trên control plane node, hoặc xem log trong `/var/log/pods/` để tìm lỗi.
   
## Xem các tùy chọn của kube-apiserver - Thủ công
- Với cluster không dùng kubeadm, bạn có thể kiểm tra các tùy chọn bằng cách xem file kube-apiserver.service
  ```
  $ cat /etc/systemd/system/kube-apiserver.service
  ```
  
  ![kube-apiserver3](../../images/kube-apiserver3.PNG)
   
- Bạn cũng có thể xem tiến trình đang chạy và các tùy chọn đang có hiệu lực bằng cách liệt kê tiến trình trên master node và tìm kube-apiserver.
  ```
  $ ps -aux | grep kube-apiserver
  ```
  ![kube-apiserver4](../../images/kube-apiserver4.PNG)

Tài liệu tham khảo K8s:
- https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/
- https://kubernetes.io/docs/concepts/overview/components/
- https://kubernetes.io/docs/concepts/overview/kubernetes-api/
- https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/
- https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/
