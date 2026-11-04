# Kube Controller Manager

  - Xem [Video hướng dẫn](https://kodekloud.com/topic/kube-controller-manager/)
  
Trong phần này, chúng ta sẽ tìm hiểu kube-controller-manager.

#### Kube Controller Manager quản lý các controller khác nhau trong Kubernetes.
- Theo thuật ngữ Kubernetes, controller là một tiến trình liên tục theo dõi trạng thái của các thành phần trong hệ thống và làm việc để đưa toàn bộ hệ thống về trạng thái hoạt động mong muốn (desired state).

> 💡 **Bổ sung:** Cơ chế này gọi là **control loop** (hay reconciliation loop): controller liên tục *quan sát* trạng thái hiện tại qua kube-apiserver → *so sánh* với trạng thái mong muốn (khai báo trong `spec`) → *hành động* để giảm khác biệt. Controller không truy cập etcd trực tiếp mà luôn đi qua kube-apiserver.

## Node Controller
   - Chịu trách nhiệm theo dõi trạng thái các node và thực hiện các hành động cần thiết để giữ cho ứng dụng tiếp tục chạy.
  
   ![node-controller](../../images/node-controller.PNG)

> 💡 **Bổ sung:** Node controller kiểm tra trạng thái node theo chu kỳ `--node-monitor-period` (mặc định 5 giây). Nếu node không gửi heartbeat trong khoảng `--node-monitor-grace-period` (mặc định 40 giây ở các bản cũ, các bản mới đã tăng lên 50 giây), node bị đánh dấu `NotReady`/`Unknown`. Tùy chọn `--pod-eviction-timeout` (5 phút) trong bài giảng đã bị loại bỏ; hiện nay việc đuổi Pod khỏi node lỗi dùng cơ chế **taint-based eviction**: node lỗi bị gắn taint `node.kubernetes.io/not-ready` hoặc `node.kubernetes.io/unreachable`, và mỗi Pod mặc định có toleration với `tolerationSeconds: 300` cho các taint này — nghĩa là sau khoảng 5 phút Pod sẽ bị evict.
   
## Replication Controller
   - Chịu trách nhiệm theo dõi trạng thái của ReplicaSet và đảm bảo luôn có đủ số lượng Pod mong muốn trong tập đó tại mọi thời điểm.
   
   ![replication-controller](../../images/replication-controller.PNG)
   
## Các controller khác
   - Còn rất nhiều controller tương tự khác trong Kubernetes
     
   ![other-controllers](../../images/other-controllers.PNG)
   
   
  ## Cài đặt Kube-Controller-Manager
  - Khi bạn cài kube-controller-manager, các controller khác nhau cũng được cài kèm.
  - Tải binary kube-controller-manager từ trang phát hành của Kubernetes. Ví dụ: bạn có thể tải kube-controller-manager v1.13.0 tại đây [kube-controller-manager](https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-controller-manager)
    ```
    $ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-controller-manager
    ```
  - Mặc định tất cả controller đều được bật, nhưng bạn có thể chọn chỉ bật một số controller cụ thể trong **`kube-controller-manager.service`**
    ```
    $ cat /etc/systemd/system/kube-controller-manager.service
    ```
    ![kube-controller-manager](../../images/kube-controller-manager.PNG)

> 💡 **Bổ sung:** Việc bật/tắt controller được cấu hình qua tùy chọn `--controllers` (giá trị mặc định `*` = bật tất cả controller mặc định; dùng `-<tên>` để tắt một controller, ví dụ `--controllers=*,-ttl`). Khi một controller "không hoạt động" (ví dụ scale Deployment mà không có Pod mới được tạo), hãy kiểm tra Pod kube-controller-manager trong `kube-system` đầu tiên.
    
## Xem kube-controller-manager - kubeadm
- kubeadm triển khai kube-controller-manager dưới dạng một Pod trong namespace kube-system
  ```
  $ kubectl get pods -n kube-system
  ```
  ![kube-controller-manager0](../../images/kube-controller-manager0.PNG)
  
## Xem các tùy chọn của kube-controller-manager - kubeadm
- Bạn có thể xem các tùy chọn trong định nghĩa Pod nằm tại **`/etc/kubernetes/manifests/kube-controller-manager.yaml`**
  ```
  $ cat /etc/kubernetes/manifests/kube-controller-manager.yaml
  ```
  ![kube-controller-manager1](../../images/kube-controller-manager1.PNG)
  
## Xem các tùy chọn của kube-controller-manager - Thủ công
- Với cluster không dùng kubeadm, bạn có thể kiểm tra các tùy chọn bằng cách xem file **`kube-controller-manager.service`**
  ```
  $ cat /etc/systemd/system/kube-controller-manager.service
  ```
  ![kube-controller-manager2](../../images/kube-controller-manager2.PNG)
  
- Bạn cũng có thể xem tiến trình đang chạy và các tùy chọn đang có hiệu lực bằng cách liệt kê tiến trình trên master node và tìm kube-controller-manager.
  ```
  $ ps -aux | grep kube-controller-manager
  ```
  ![kube-controller-manager3](../../images/kube-controller-manager3.PNG)
  
Tài liệu tham khảo K8s:
- https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/
- https://kubernetes.io/docs/concepts/overview/components/
   
     
