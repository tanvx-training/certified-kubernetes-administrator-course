# Kube Scheduler
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/kube-scheduler/)

Trong phần này, chúng ta sẽ tìm hiểu kube-scheduler.

#### kube-scheduler chịu trách nhiệm lập lịch (schedule) Pod lên các node.  
- kube-scheduler chỉ chịu trách nhiệm quyết định Pod nào chạy trên node nào. Nó không thực sự đặt Pod lên node — đó là công việc của **`kubelet`**.

  ![kube-scheduler1](../../images/kube-scheduler1.PNG)
  
#### Tại sao cần có Scheduler?

  ![kube-scheduler2](../../images/kube-scheduler2.PNG)

> 💡 **Bổ sung:** Scheduler chọn node qua hai bước chính:
> 1. **Filtering (lọc):** loại bỏ các node không phù hợp — ví dụ không đủ CPU/memory theo resource requests của Pod, có taint mà Pod không tolerate, không khớp `nodeSelector`/node affinity...
> 2. **Scoring (chấm điểm):** chấm điểm các node còn lại (ví dụ node còn nhiều tài nguyên trống hơn sau khi đặt Pod sẽ được điểm cao hơn) và chọn node có điểm cao nhất.
>
> Các bước này được hiện thực bằng các plugin trong *scheduling framework* và có thể tùy chỉnh qua `KubeSchedulerConfiguration` (sẽ học ở phần Scheduling). Nếu không có scheduler chạy (hoặc scheduler lỗi), Pod mới sẽ nằm mãi ở trạng thái `Pending` — đây là tình huống troubleshooting rất hay gặp.
    
## Cài đặt kube-scheduler - Thủ công
- Tải binary kube-scheduler từ trang phát hành của Kubernetes [kube-scheduler](https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-scheduler). Ví dụ: để tải kube-scheduler v1.13.0, chạy lệnh dưới đây.
  ```
  $ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-scheduler
  ```
- Giải nén
- Chạy nó dưới dạng service

  ![kube-scheduler3](../../images/kube-scheduler3.PNG)
  
## Xem các tùy chọn của kube-scheduler - kubeadm
- Nếu bạn dựng cluster bằng công cụ kubeadm, kubeadm sẽ triển khai kube-scheduler dưới dạng một Pod trong namespace kube-system trên master node.
  ```
  $ kubectl get pods -n kube-system
  ```
- Bạn có thể xem các tùy chọn của kube-scheduler trong file định nghĩa Pod nằm tại **`/etc/kubernetes/manifests/kube-scheduler.yaml`**
  ```
  $ cat /etc/kubernetes/manifests/kube-scheduler.yaml
  ```
  ![kube-scheduler4](../../images/kube-scheduler4.PNG)
  
- Bạn cũng có thể xem tiến trình đang chạy và các tùy chọn đang có hiệu lực bằng cách liệt kê tiến trình trên master node và tìm kube-scheduler.
  ``` 
  $ ps -aux | grep kube-scheduler
  ```
  ![kube-scheduler5](../../images/kube-scheduler5.PNG)
  
  Tài liệu tham khảo K8s:
  - https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/
  - https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/
  - https://kubernetes.io/docs/concepts/overview/components/
  - https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/
    
