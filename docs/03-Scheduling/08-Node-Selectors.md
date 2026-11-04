# Node Selectors
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/node-selectors/)

Trong phần này, chúng ta sẽ tìm hiểu về Node Selectors trong Kubernetes. Giả sử cluster có 3 node: 2 node nhỏ và 1 node lớn nhiều tài nguyên hơn. Ta muốn Pod xử lý dữ liệu nặng chỉ chạy trên node lớn.

#### Ta thêm thuộc tính mới tên là `nodeSelector` vào phần `spec` và chỉ định label.
- Scheduler dùng các label này để so khớp và xác định node phù hợp để đặt Pod lên.
  ```
  apiVersion: v1
  kind: Pod
  metadata:
   name: myapp-pod
  spec:
   containers:
   - name: data-processor
     image: data-processor
   nodeSelector:
    size: Large
  ```
![nsel](../../images/nsel.PNG)
  
- Gắn label cho node (phải làm trước khi tạo Pod)

  Cú pháp
  ```
  $ kubectl label nodes <node-name> <label-key>=<label-value>
  ```
  Ví dụ
  ```
  $ kubectl label nodes node-1 size=Large
  ```
  
![ln](../../images/ln.PNG)
  
- Tạo Pod từ file định nghĩa
  ```
  apiVersion: v1
  kind: Pod
  metadata:
   name: myapp-pod
  spec:
   containers:
   - name: data-processor
     image: data-processor
   nodeSelector:
    size: Large
  ```
  ```
  $ kubectl create -f pod-definition.yml
  ```
  
![nsel](../../images/nsel.PNG)
  
## Node Selector - Hạn chế
- Ở đây ta chỉ dùng một label và selector đơn giản để đạt được mục tiêu. Nhưng nếu yêu cầu phức tạp hơn thì sao? Ví dụ: "đặt Pod lên node Large **hoặc** Medium", hay "đặt Pod lên bất kỳ node nào **không phải** Small". `nodeSelector` không biểu diễn được các điều kiện như vậy.
  
![nsl](../../images/nsl.PNG)
 
- Để giải quyết, ta có **`Node Affinity và Anti Affinity`**

> 💡 **Bổ sung:**
> - Nếu khai báo nhiều cặp key/value trong `nodeSelector`, node phải có **tất cả** các label đó (điều kiện AND).
> - Nếu không node nào khớp label, Pod sẽ ở trạng thái `Pending`.
> - Mỗi node đã có sẵn một số label do Kubernetes tự đặt, ví dụ `kubernetes.io/hostname`, `kubernetes.io/os`, `kubernetes.io/arch`, `topology.kubernetes.io/zone`. Có thể dùng chúng trong `nodeSelector`, ví dụ `kubernetes.io/hostname: node01`. Xem label của node bằng `kubectl get nodes --show-labels`.
> - `nodeSelector` và node affinity có thể dùng cùng lúc; khi đó node phải thỏa mãn **cả hai**.
  
#### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector





