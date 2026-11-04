# ReplicaSets
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/replicasets/)

Trong phần này, chúng ta sẽ tìm hiểu:
- Replication Controller
- ReplicaSet

#### Controller là "bộ não" của Kubernetes

> 💡 **Bổ sung:** Controller là các tiến trình chạy vòng lặp điều hòa (reconciliation loop): liên tục theo dõi trạng thái thực tế của các object trong cluster và thực hiện hành động để đưa nó về trạng thái mong muốn (desired state) mà bạn khai báo. Các controller tích hợp sẵn (ReplicaSet, Deployment, Node, Endpoint...) đều chạy bên trong `kube-controller-manager`.

## Replica là gì và vì sao cần Replication Controller?

  ![rc](../../images/rc.PNG)
  
  ![rc1](../../images/rc1.PNG)

> 💡 **Bổ sung:** Replica là các bản sao giống hệt nhau của một Pod. Replication Controller giúp:
> - **Tính sẵn sàng cao (high availability):** nếu một Pod chết, controller tự tạo Pod mới thay thế, kể cả khi bạn chỉ chạy 1 replica.
> - **Cân bằng tải và mở rộng (load balancing & scaling):** chạy nhiều Pod để chia tải, và các Pod có thể nằm trên nhiều node khác nhau của cluster.
  
## Sự khác nhau giữa ReplicaSet và Replication Controller
- **`Replication Controller`** là công nghệ cũ, đang được thay thế bởi **`ReplicaSet`**.
- **`ReplicaSet`** là cách mới để thiết lập replication.

> 💡 **Bổ sung:** Điểm khác biệt chính:
> - `ReplicationController` thuộc API `v1`; `ReplicaSet` thuộc API `apps/v1`.
> - `ReplicaSet` bắt buộc có trường `selector` và hỗ trợ selector dạng tập hợp (`matchExpressions` với `In`, `NotIn`, `Exists`...), còn `ReplicationController` chỉ hỗ trợ so khớp bằng đẳng thức.
> - Trong thực tế bạn hầu như không tạo ReplicaSet trực tiếp mà dùng **Deployment** (Deployment sẽ tự quản lý ReplicaSet và hỗ trợ rolling update/rollback).

## Tạo Replication Controller

## File định nghĩa Replication Controller
  
   ![rc2](../../images/rc2.PNG)
  
```
    apiVersion: v1
    kind: ReplicationController
    metadata:
      name: myapp-rc
      labels:
        app: myapp
        type: front-end
    spec:
     template:
        metadata:
          name: myapp-pod
          labels:
            app: myapp
            type: front-end
        spec:
         containers:
         - name: nginx-container
           image: nginx
     replicas: 3
```
  - Tạo Replication Controller
    ```
    $ kubectl create -f rc-definition.yaml
    ```
  - Liệt kê tất cả Replication Controller
    ```
    $ kubectl get replicationcontroller
    ```
  - Liệt kê các Pod được tạo bởi Replication Controller
    ```
    $ kubectl get pods
    ```
    ![rc3](../../images/rc3.PNG)
    
## Tạo ReplicaSet
  
## File định nghĩa ReplicaSet

   ![rs](../../images/rs.PNG)

```
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: myapp-replicaset
      labels:
        app: myapp
        type: front-end
    spec:
     template:
        metadata:
          name: myapp-pod
          labels:
            app: myapp
            type: front-end
        spec:
         containers:
         - name: nginx-container
           image: nginx
     replicas: 3
     selector:
       matchLabels:
        type: front-end
 ```
#### So với Replication Controller, ReplicaSet bắt buộc phải có phần định nghĩa selector.

> 💡 **Bổ sung:** Label trong `spec.selector.matchLabels` **phải khớp** với label trong `spec.template.metadata.labels`, nếu không API server sẽ từ chối với lỗi `selector does not match template labels`. Selector cũng giúp ReplicaSet "nhận nuôi" cả những Pod đã tồn tại sẵn (không do nó tạo) nếu các Pod đó có label khớp - vì vậy cần cẩn thận để không trùng label giữa các ứng dụng.
   
  - Tạo ReplicaSet
    ```
    $ kubectl create -f replicaset-definition.yaml
    ```
  - Liệt kê tất cả ReplicaSet
    ```
    $ kubectl get replicaset
    ```
  - Liệt kê các Pod được tạo bởi ReplicaSet
    ```
    $ kubectl get pods
    ```
   
    ![rs1](../../images/rs1.PNG)
    
## Labels và Selectors
#### Labels và Selectors dùng để làm gì? Vì sao chúng ta gắn label cho Pod và các object trong Kubernetes?

  ![labels](../../images/labels.PNG)

> 💡 **Bổ sung:** Label là các cặp key/value gắn vào object để phân loại. Selector là bộ lọc dựa trên label. ReplicaSet dùng selector để biết nó cần giám sát những Pod nào - khi có hàng trăm Pod trong cluster, đây là cách để ReplicaSet xác định "Pod của mình". Bạn có thể thử lọc bằng tay: `kubectl get pods -l type=front-end` hoặc `kubectl get pods --selector app=myapp`.
  
## Cách scale ReplicaSet
- Có nhiều cách để scale một ReplicaSet
  - Cách thứ nhất là cập nhật số replicas trong file định nghĩa replicaset-definition.yaml, ví dụ replicas: 6, rồi chạy lệnh
 ```
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: myapp-replicaset
      labels:
        app: myapp
        type: front-end
    spec:
     template:
        metadata:
          name: myapp-pod
          labels:
            app: myapp
            type: front-end
        spec:
         containers:
         - name: nginx-container
           image: nginx
     replicas: 6
     selector:
       matchLabels:
        type: front-end
```

  ```
  $ kubectl apply -f replicaset-definition.yaml
  ```
  - Cách thứ hai là dùng lệnh **`kubectl scale`**.
  ```
  $ kubectl scale --replicas=6 -f replicaset-definition.yaml
  ```
  - Cách thứ ba là dùng lệnh **`kubectl scale`** kèm loại resource và tên
  ```
  $ kubectl scale --replicas=6 replicaset myapp-replicaset
  ```
  ![rs2](../../images/rs2.PNG)

> 💡 **Bổ sung:** Lưu ý: dùng `kubectl scale` (cách 2 và 3) **không** cập nhật số `replicas` trong file YAML của bạn - file vẫn giữ giá trị cũ (ví dụ `replicas: 3`). Nếu sau đó bạn `kubectl apply -f` lại file đó, số replica sẽ quay về giá trị trong file. Ngoài ra, còn có thể scale tự động theo tải bằng HorizontalPodAutoscaler (HPA) - xem lệnh `kubectl autoscale` (thường áp dụng cho Deployment).

#### Tài liệu tham khảo K8s:
- https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/
- https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/
