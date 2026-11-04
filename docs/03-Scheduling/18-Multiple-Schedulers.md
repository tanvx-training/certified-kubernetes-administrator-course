# Multiple Schedulers (Nhiều scheduler)
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/multiple-schedulers/)

Trong phần này, chúng ta sẽ tìm hiểu về việc chạy nhiều scheduler

## Custom Schedulers
- Default scheduler phân bổ Pod đều lên các node, có xét đến taint/toleration, node affinity... Nhưng nếu ứng dụng của bạn cần các bước kiểm tra riêng trước khi đặt Pod lên node, bạn có thể tự viết scheduler, đóng gói và triển khai nó làm default scheduler hoặc làm scheduler bổ sung.
- Kubernetes cluster của bạn có thể chạy nhiều scheduler cùng lúc. Khi tạo Pod hoặc Deployment, bạn có thể chỉ định Pod được lập lịch bởi scheduler nào.
- Mỗi scheduler phải có một tên riêng. Default scheduler có tên `default-scheduler`; tên này được cấu hình trong file cấu hình scheduler (`KubeSchedulerConfiguration`), ví dụ:
  ```
  apiVersion: kubescheduler.config.k8s.io/v1
  kind: KubeSchedulerConfiguration
  profiles:
  - schedulerName: my-scheduler
  ```

  ![ms](../../images/ms.PNG)
  
## Triển khai scheduler bổ sung
- Tải file binary (chạy dưới dạng service). Mỗi scheduler dùng một file cấu hình riêng chứa tên scheduler, truyền vào qua option `--config`.
  ```
  $ wget https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-scheduler
  ```
  ![das](../../images/das.PNG)
  
## Triển khai scheduler bổ sung - kubeadm
- Trong cluster kubeadm, các thành phần control plane chạy dưới dạng Pod. Ta có thể triển khai scheduler bổ sung dưới dạng Pod (hoặc Deployment), dùng image `kube-scheduler`, trỏ `--kubeconfig` tới `/etc/kubernetes/scheduler.conf` (chứa thông tin xác thực để kết nối tới kube-apiserver) và `--config` tới file cấu hình riêng của scheduler.
- Option `leaderElection` trong file cấu hình dùng khi chạy nhiều bản sao của cùng một scheduler trên nhiều master node (môi trường HA): chỉ một bản sao được "bầu" làm leader và hoạt động tại một thời điểm.
   
  ![dask](../../images/dask.PNG)
  
  - Tạo Pod scheduler
    ```
    $ kubectl create -f my-custom-scheduler.yaml
    ```
  
## Xem các Scheduler
- Liệt kê các Pod scheduler
  ```
  $ kubectl get pods -n kube-system
  ```

## Dùng Custom Scheduler
- Tạo file định nghĩa Pod, thêm trường mới **`schedulerName`** và chỉ định tên của scheduler mới. Nếu scheduler không được cấu hình đúng hoặc không chạy, Pod sẽ ở trạng thái `Pending`.
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx
  spec:
    containers:
    - image: nginx
      name: nginx
    schedulerName: my-custom-scheduler
  ```
  ![cs](../../images/cs.png)
  
- Tạo Pod từ file định nghĩa
  ```
  $ kubectl create -f pod-definition.yaml
  ```
- Liệt kê các Pod
  ```
  $ kubectl get pods
  ```

## Xem Events
- Để biết Pod được lập lịch bởi scheduler nào, xem events: tìm event có `REASON` là `Scheduled`; thêm `-o wide` để thấy cột `SOURCE` hiển thị tên scheduler
  ```
  $ kubectl get events
  ```
  ![cs1](../../images/cs1.PNG)
  
## Xem log của Scheduler
- Xem log của scheduler
  ```
  $ kubectl logs my-custom-scheduler -n kube-system
  ```
  ![cs2](../../images/cs2.PNG)

> 💡 **Bổ sung (cập nhật cho Kubernetes hiện tại):**
> - URL tải binary ở trên (`storage.googleapis.com/kubernetes-release`) là đường dẫn cũ; hiện nay binary được phân phối qua `https://dl.k8s.io/release/<version>/bin/linux/amd64/kube-scheduler`.
> - Các option cũ như `--scheduler-name` và `--lock-object-name` đã bị loại bỏ; tên scheduler giờ được khai báo trong `profiles[].schedulerName` của file `KubeSchedulerConfiguration` (API `kubescheduler.config.k8s.io/v1`, GA từ v1.25).
> - Ví dụ file cấu hình đầy đủ hơn cho scheduler bổ sung, thường được lưu trong ConfigMap và mount vào Pod scheduler:
>   ```
>   apiVersion: kubescheduler.config.k8s.io/v1
>   kind: KubeSchedulerConfiguration
>   profiles:
>   - schedulerName: my-scheduler
>   leaderElection:
>     leaderElect: false
>   ```
> - Khi triển khai scheduler dưới dạng Pod/Deployment (như trong tài liệu chính thức), cần tạo ServiceAccount và gán ClusterRole `system:kube-scheduler` (và `system:volume-scheduler`) cho nó để scheduler có quyền đọc Pod/node và tạo Binding.
> - Xem nhanh Pod được lập lịch bởi scheduler nào: `kubectl get events -o wide` hoặc `kubectl describe pod <tên>` (phần `Events`, cột `From`).
> - Thay vì chạy nhiều binary scheduler riêng biệt, từ v1.18 có thể dùng **scheduler profiles**: một tiến trình kube-scheduler phục vụ nhiều `schedulerName` với cấu hình plugin khác nhau (xem phần [Configuring Kubernetes Schedulers](20-Configuring-Kubernetes-Schedulers.md)).
  
#### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/
  
