# Taints và Tolerations
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/taints-and-tolerations-2/)
  
Trong phần này, chúng ta sẽ tìm hiểu về taint (vết đánh dấu trên node) và toleration (khả năng "chịu đựng" taint của Pod).
- Đây là quan hệ giữa Pod và node, và cách bạn giới hạn Pod nào được đặt lên node nào.

#### Taints và Tolerations dùng để đặt ra giới hạn về việc Pod nào được phép lập lịch lên một node.
- Chỉ những Pod có toleration phù hợp với taint trên node mới được lập lịch lên node đó.

  ![tandt](../../images/tandt.PNG)
  
## Taints
- Dùng lệnh **`kubectl taint nodes`** để đặt taint cho node.

  Cú pháp
  ```
  $ kubectl taint nodes <node-name> key=value:taint-effect
  ```
 
  Ví dụ
  ```
  $ kubectl taint nodes node1 app=blue:NoSchedule
  ```
  
- Taint effect quy định điều gì sẽ xảy ra với các Pod nếu chúng không tolerate (chịu được) taint đó.
- Có 3 loại taint effect
  - **`NoSchedule`**: Pod không tolerate taint sẽ không được lập lịch lên node. Các Pod đang chạy trên node không bị ảnh hưởng.
  - **`PreferNoSchedule`**: scheduler sẽ *cố gắng* tránh đặt Pod lên node, nhưng không đảm bảo (đây là ràng buộc "mềm").
  - **`NoExecute`**: Pod mới không tolerate taint sẽ không được lập lịch lên node, đồng thời các Pod đang chạy trên node mà không tolerate taint sẽ bị evict (đuổi) khỏi node.
  
  ![tn](../../images/tn.PNG)
  
## Tolerations
   - Toleration được thêm vào Pod bằng cách khai báo phần **`tolerations`** trong file định nghĩa Pod. Các giá trị `key`, `value`, `effect` phải khớp với taint đã đặt trên node.
     ```
     apiVersion: v1
     kind: Pod
     metadata:
      name: myapp-pod
     spec:
      containers:
      - name: nginx-container
        image: nginx
      tolerations:
      - key: "app"
        operator: "Equal"
        value: "blue"
        effect: "NoSchedule"
     ```
    
  ![tp](../../images/tp.PNG)
    

#### Taints và Tolerations không bảo Pod phải đến một node cụ thể. Thay vào đó, chúng bảo node chỉ chấp nhận những Pod có toleration nhất định.
- Nghĩa là Pod có toleration vẫn có thể được đặt lên một node khác không có taint. Nếu muốn ép Pod chạy trên một node cụ thể, cần dùng node affinity (xem các phần sau).
- Scheduler không bao giờ đặt Pod ứng dụng lên master node. Lý do là khi cluster được dựng, một taint được tự động đặt trên master (control plane) node để ngăn Pod chạy trên đó. Best practice là không chạy workload ứng dụng trên master node.
- Để xem taint này, chạy lệnh sau
  ```
  $ kubectl describe node kubemaster |grep Taint
  ```
 
 ![tntm](../../images/tntm.PNG)
  
> 💡 **Bổ sung:**
> - Xóa taint: thêm dấu `-` vào cuối, ví dụ `kubectl taint nodes node1 app=blue:NoSchedule-` (hoặc chỉ `kubectl taint nodes node1 app-` để xóa mọi taint có key `app`).
> - Toán tử `operator`:
>   - `Equal` (mặc định): khớp khi `key`, `value` và `effect` đều giống taint.
>   - `Exists`: chỉ cần khớp `key` (không khai báo `value`). Toleration `operator: Exists` mà không có `key` sẽ tolerate **mọi** taint.
>   - Nếu bỏ trống `effect`, toleration khớp với mọi effect của key đó.
> - Với `NoExecute`, có thể khai báo `tolerationSeconds` để Pod được ở lại node thêm một khoảng thời gian trước khi bị evict:
>   ```
>   tolerations:
>   - key: "node.kubernetes.io/unreachable"
>     operator: "Exists"
>     effect: "NoExecute"
>     tolerationSeconds: 300
>   ```
>   Kubernetes tự động thêm các toleration `node.kubernetes.io/not-ready` và `node.kubernetes.io/unreachable` với `tolerationSeconds: 300` cho Pod — đó là lý do Pod bị chuyển đi khoảng 5 phút sau khi node gặp sự cố.
> - Từ Kubernetes v1.24, taint của control plane node do kubeadm đặt là `node-role.kubernetes.io/control-plane:NoSchedule` (taint cũ `node-role.kubernetes.io/master` đã bị loại bỏ).
> - Node controller cũng tự đặt taint theo tình trạng node, ví dụ `node.kubernetes.io/not-ready`, `node.kubernetes.io/unreachable`, `node.kubernetes.io/disk-pressure`, `node.kubernetes.io/memory-pressure`, `node.kubernetes.io/unschedulable` (khi node bị `cordon`).
> - Xem taint của tất cả node nhanh: `kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints`.
     
#### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/

