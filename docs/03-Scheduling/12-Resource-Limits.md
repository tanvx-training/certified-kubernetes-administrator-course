# Resource Limits (Giới hạn tài nguyên)
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/resource-limits/)
  
Trong phần này, chúng ta sẽ tìm hiểu về Resource Limits

#### Hãy xem xét một Kubernetes cluster gồm 3 node.
- Mỗi node có một lượng tài nguyên CPU, Memory và Disk nhất định.
- Mỗi Pod cần một lượng tài nguyên để chạy. kube-scheduler quyết định Pod sẽ chạy trên node nào dựa trên lượng tài nguyên Pod yêu cầu và lượng tài nguyên còn trống trên node.
- Nếu không node nào còn đủ tài nguyên, Kubernetes sẽ hoãn lập lịch Pod. Bạn sẽ thấy Pod ở trạng thái `Pending`. Xem phần events, bạn sẽ thấy lý do là không đủ CPU (`Insufficient cpu`).
  
  ![rl](../../images/rl.PNG)
  
## Resource Requirements (Yêu cầu tài nguyên)
- Theo khóa học gốc, mặc định K8s giả định một Pod hoặc container trong Pod cần **`0.5`** CPU và **`256Mi`** memory. Lượng tài nguyên tối thiểu mà container yêu cầu được gọi là **`Resource Request` của container**. (Xem lưu ý bên dưới: giá trị mặc định này chỉ có khi namespace được cấu hình LimitRange.)
  
  ![rr](../../images/rr.PNG)
  
- Nếu ứng dụng trong Pod cần nhiều hơn mức mặc định, bạn cần khai báo trong file định nghĩa Pod.

  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: simple-webapp-color
    labels:
      name: simple-webapp-color
  spec:
   containers:
   - name: simple-webapp-color
     image: simple-webapp-color
     ports:
      - containerPort:  8080
     resources:
       requests:
        memory: "1Gi"
        cpu: "1"
  ```
  ![rr-pod](../../images/rr-pod.PNG) 

- Đơn vị:
  - CPU: `1` = 1 vCPU (1 AWS vCPU, 1 GCP Core, 1 Azure Core hoặc 1 Hyperthread). Có thể dùng số thập phân như `0.1` hoặc dạng milli như `100m` (= 0.1 CPU). Giá trị nhỏ nhất là `1m`.
  - Memory: `Mi`, `Gi` (lũy thừa của 2: 1 Mi = 1.048.576 byte) khác với `M`, `G` (lũy thừa của 10: 1 M = 1.000.000 byte).
   
## Resources - Limits
- Theo khóa học gốc, mặc định K8s đặt resource limit là 1 CPU và 512Mi memory (tương tự như trên, chỉ đúng khi có LimitRange).
  
  ![rsl](../../images/rsl.PNG)
  
- Bạn có thể khai báo resource limit trong file định nghĩa Pod.
  
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: simple-webapp-color
    labels:
      name: simple-webapp-color
  spec:
   containers:
   - name: simple-webapp-color
     image: simple-webapp-color
     ports:
      - containerPort:  8080
     resources:
       requests:
        memory: "1Gi"
        cpu: "1"
       limits:
         memory: "2Gi"
         cpu: "2"
  ```
  ![rsl1](../../images/rsl1.PNG)
  
#### Lưu ý: Requests và Limits được đặt cho từng container trong Pod.
  
## Vượt quá Limits
- Điều gì xảy ra khi Pod cố dùng tài nguyên vượt quá limit?
  - **CPU**: container không thể dùng nhiều CPU hơn limit — hệ thống sẽ throttle (bóp) CPU. Container không bị kill vì CPU.
  - **Memory**: container *có thể* tạm dùng nhiều memory hơn limit, nhưng nếu liên tục vượt limit, container sẽ bị kill với lý do `OOMKilled` (Out Of Memory).

   ![el](../../images/el.PNG)

> 💡 **Bổ sung (cập nhật cho Kubernetes hiện tại):**
> - **Kubernetes không có giá trị request/limit mặc định.** Nếu không khai báo, container không có request (scheduler coi như 0) và không có limit (có thể dùng toàn bộ tài nguyên của node). Các giá trị mặc định như 0.5 CPU/256Mi hay 1 CPU/512Mi trong video chỉ áp dụng khi admin tạo **LimitRange** trong namespace:
>   ```
>   apiVersion: v1
>   kind: LimitRange
>   metadata:
>     name: cpu-resource-constraint
>   spec:
>     limits:
>     - default:          # limit mặc định
>         cpu: 500m
>       defaultRequest:   # request mặc định
>         cpu: 500m
>       max:
>         cpu: "1"
>       min:
>         cpu: 100m
>       type: Container
>   ```
>   LimitRange chỉ áp dụng cho Pod được tạo **sau** khi LimitRange tồn tại.
> - Nếu chỉ đặt `limits` mà không đặt `requests`, Kubernetes tự đặt `requests` bằng `limits`.
> - **ResourceQuota** giới hạn tổng tài nguyên (và số lượng đối tượng) trong một namespace, ví dụ `requests.cpu: 4`, `limits.memory: 10Gi`, `pods: 10`. Khi namespace có quota cho cpu/memory, mọi Pod phải khai báo request/limit tương ứng (hoặc được LimitRange điền sẵn), nếu không sẽ bị từ chối.
> - **Bốn trường hợp CPU thường gặp:** không request/không limit (một Pod có thể chiếm hết CPU); không request/có limit (request tự lấy bằng limit); có request/có limit; và có request/không limit — thường là cấu hình hợp lý nhất cho CPU, vì Pod được đảm bảo phần request và có thể dùng thêm CPU nhàn rỗi. Với memory thì khác: memory không thể "thu hồi" như CPU, cách duy nhất để giải phóng là kill Pod.
> - **QoS class** (xem bằng `kubectl get pod <tên> -o jsonpath='{.status.qosClass}'`):
>   - `Guaranteed`: mọi container có request = limit cho cả CPU và memory.
>   - `Burstable`: có ít nhất một request/limit nhưng không đạt điều kiện Guaranteed.
>   - `BestEffort`: không container nào khai báo request/limit. Pod loại này bị evict đầu tiên khi node thiếu tài nguyên.
> - Xem lượng tài nguyên đã được cấp phát trên node: `kubectl describe node <tên>` (phần `Allocated resources`). Xem mức sử dụng thực tế: `kubectl top pod` / `kubectl top node` (cần cài metrics-server).
> - Tạo nhanh khi thi: `kubectl set resources deployment <tên> --requests=cpu=100m,memory=128Mi --limits=cpu=200m,memory=256Mi`.
> - Từ Kubernetes v1.33 (beta, bật mặc định), có thể thay đổi resources của Pod đang chạy mà không cần tạo lại (in-place Pod resize) thông qua subresource `resize`. Trong lab/thi, cách an toàn vẫn là xóa và tạo lại Pod.
  
#### Tài liệu tham khảo K8s:
- https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
- https://kubernetes.io/docs/concepts/policy/limit-range/
- https://kubernetes.io/docs/concepts/policy/resource-quotas/
- https://kubernetes.io/docs/concepts/workloads/pods/pod-qos/
