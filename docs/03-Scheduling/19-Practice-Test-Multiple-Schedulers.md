# Practice Test - Multiple Schedulers (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-multiple-schedulers/)
  
Lời giải cho bài practice test - Multiple Schedulers
- Chạy lệnh 'kubectl get pods --namespace=kube-system' (xác định tên Pod của default scheduler)
  
  <details>

  ```
  $ kubectl get pods --namespace=kube-system
  ```
  </details>

- Chạy lệnh 'kubectl describe pod kube-scheduler-controlplane --namespace=kube-system' (xác định image mà default scheduler đang dùng)

  <details>

  ```
  $ kubectl describe pod kube-scheduler-controlplane --namespace=kube-system
  ```
  </details>
  
- Dùng lệnh imperative để tạo ConfigMap với option --from-file (ConfigMap này chứa file cấu hình `KubeSchedulerConfiguration` của scheduler mới, sẽ được mount vào Pod scheduler)

  <details>

  ```
  $ kubectl create -n kube-system configmap my-scheduler-config --from-file=/root/my-scheduler-config.yaml
  ```
  </details>

- Dùng file /root/my-scheduler.yaml để tạo scheduler của riêng bạn với image đúng (thường là cùng image với default scheduler đã xác định ở câu 2).


  <details>

  ```
  $ kubectl create -f /root/my-scheduler.yaml
  ```
  </details>

- Đặt thuộc tính schedulerName trong spec của Pod thành tên của scheduler mới. File nằm ở /root/nginx-pod.yaml
  
  <details>

  ```
  master $ grep schedulerName /root/nginx-pod.yaml
  schedulerName: my-scheduler
  
  $ kubectl create -f /root/nginx-pod.yaml
  ```
  </details>

> 💡 **Mẹo:**
> - Tên trong `schedulerName` của Pod phải trùng khớp với `profiles[].schedulerName` trong file `my-scheduler-config.yaml`.
> - `schedulerName` nằm trong `spec` (cùng cấp với `containers`), và không thể sửa trên Pod đã tạo — phải xóa và tạo lại Pod.
> - Nếu Pod `Pending`, kiểm tra Pod `my-scheduler` có ở trạng thái `Running` không và xem log của nó: `kubectl logs my-scheduler -n kube-system`.
> - Xác nhận Pod được lập lịch bởi scheduler nào: `kubectl get events -o wide | grep Scheduled`.


