# Manual Scheduling (Lập lịch thủ công)
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/manual-scheduling/)
  
Trong phần này, chúng ta sẽ tìm hiểu cách **`lập lịch thủ công`** (manually scheduling) một **`Pod`** lên một node.

## Scheduling hoạt động như thế nào
- Bạn sẽ làm gì nếu cluster không có scheduler?
  - Mỗi Pod đều có một trường tên là `nodeName`, mặc định không được thiết lập. Thông thường bạn không khai báo trường này khi viết file manifest, Kubernetes sẽ tự động điền vào.
  - Scheduler duyệt qua tất cả các Pod và tìm những Pod chưa có thuộc tính `nodeName` — đó là những Pod cần được lập lịch. Sau đó scheduler chạy thuật toán để chọn ra node phù hợp.
  - Khi đã xác định được node, scheduler gán Pod lên node đó bằng cách đặt thuộc tính `nodeName` thành tên của node, thông qua việc tạo một đối tượng Binding.
    ```
    apiVersion: v1
    kind: Pod
    metadata:
     name: nginx
     labels:
      name: nginx
    spec:
     containers:
     - name: nginx
       image: nginx
       ports:
       - containerPort: 8080
     nodeName: node02
    ```
    ![sc1](../../images/sc1.png)
    
## Không có Scheduler
  - Bạn có thể tự gán Pod vào node. Khi không có scheduler, cách đơn giản nhất để lập lịch cho Pod là đặt thuộc tính **`nodeName`** trong file định nghĩa Pod ngay lúc tạo Pod. Nếu không có scheduler và cũng không đặt `nodeName`, Pod sẽ ở trạng thái `Pending` mãi mãi.
    
    ![sc2](../../images/sc2.PNG)
    
  - Một cách khác: `nodeName` chỉ có thể đặt lúc tạo Pod, Kubernetes không cho phép sửa `nodeName` của một Pod đã tồn tại. Để gán một Pod đã tồn tại vào node, ta tạo một đối tượng Binding rồi gửi request POST tới Binding API của Pod (giống cách scheduler thực sự làm). Dữ liệu gửi đi phải ở dạng JSON tương đương với file YAML bên dưới.
    ```
    apiVersion: v1
    kind: Binding
    metadata:
      name: nginx
    target:
      apiVersion: v1
      kind: Node
      name: node02
    ```
    ```
    apiVersion: v1
    kind: Pod
    metadata:
     name: nginx
     labels:
      name: nginx
    spec:
     containers:
     - name: nginx
       image: nginx
       ports:
       - containerPort: 8080
    ```
    ![sc3](../../images/sc3.PNG)
    
> 💡 **Bổ sung:**
> - `nodeName` **bỏ qua hoàn toàn scheduler**: kubelet trên node được chỉ định sẽ nhận Pod ngay, không xét taint `NoSchedule`, node affinity hay việc node còn đủ tài nguyên hay không (nếu thiếu tài nguyên, kubelet có thể từ chối Pod với lý do như `OutOfcpu`/`OutOfmemory`). Nếu tên node không tồn tại, Pod sẽ không chạy và có thể bị xóa.
> - Cách gửi Binding thủ công (minh họa trong hình `sc3`) — dùng `kubectl proxy` hoặc `curl` với thông tin xác thực phù hợp:
>   ```
>   curl --header "Content-Type:application/json" --request POST \
>     --data '{"apiVersion":"v1","kind":"Binding","metadata":{"name":"nginx"},"target":{"apiVersion":"v1","kind":"Node","name":"node02"}}' \
>     http://$SERVER/api/v1/namespaces/default/pods/nginx/binding/
>   ```
> - Mẹo thi: nếu đề yêu cầu "schedule Pod lên node X" mà Pod đang chạy, cách nhanh nhất là `kubectl get pod <tên> -o yaml > pod.yaml`, thêm `nodeName`, rồi `kubectl replace --force -f pod.yaml` (xóa và tạo lại trong một lệnh).
> - Khi Pod `Pending` và không có sự kiện nào trong `kubectl describe pod`, hãy kiểm tra xem kube-scheduler có đang chạy không (`kubectl get pods -n kube-system`). Trên cluster dựng bằng kubeadm, scheduler là static Pod với manifest tại `/etc/kubernetes/manifests/kube-scheduler.yaml`.
    
Tài liệu tham khảo K8s:
- https://kubernetes.io/docs/reference/using-api/api-concepts/
- https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodename
    
    
   
