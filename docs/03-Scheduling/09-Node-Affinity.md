# Node Affinity
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/node-affinity-2/)
  
Trong phần này, chúng ta sẽ tìm hiểu tính năng "Node Affinity" trong Kubernetes.

#### Mục đích chính của Node Affinity là đảm bảo Pod được đặt lên những node cụ thể.
- Với **`Node Selectors`**, ta không thể dùng các biểu thức nâng cao (như OR, NOT). Node Affinity cung cấp khả năng này, đổi lại cú pháp phức tạp hơn. Hai file dưới đây cho cùng một kết quả: đặt Pod lên node có label `size=Large`.
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
  ![ns-old](../../images/ns-old.PNG)

- Dùng toán tử `In` để đặt Pod lên node có label `size` là `Large` **hoặc** `Medium`:
  ```
  apiVersion: v1
  kind: Pod
  metadata:
   name: myapp-pod
  spec:
   containers:
   - name: data-processor
     image: data-processor
   affinity:
     nodeAffinity:
       requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: size
              operator: In
              values: 
              - Large
              - Medium
  ```
  ![na](../../images/na.PNG)
  
- Dùng toán tử `NotIn` để đặt Pod lên bất kỳ node nào mà label `size` **không phải** `Small`:
  ```
  apiVersion: v1
  kind: Pod
  metadata:
   name: myapp-pod
  spec:
   containers:
   - name: data-processor
     image: data-processor
   affinity:
     nodeAffinity:
       requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: size
              operator: NotIn
              values: 
              - Small
  ```
  ![na1](../../images/na1.PNG)
  
- Dùng toán tử `Exists` để kiểm tra node có label `size` hay không (không quan tâm giá trị, nên không cần `values`):
  ```
  apiVersion: v1
  kind: Pod
  metadata:
   name: myapp-pod
  spec:
   containers:
   - name: data-processor
     image: data-processor
   affinity:
     nodeAffinity:
       requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: size
              operator: Exists
  ```
  
  ![na2](../../images/na2.PNG)
  

## Các loại Node Affinity
- Đang có
  - requiredDuringSchedulingIgnoredDuringExecution
  - preferredDuringSchedulingIgnoredDuringExecution
- Dự kiến (planned)
  - requiredDuringSchedulingRequiredDuringExecution
  - preferredDuringSchedulingRequiredDuringExecution
  
  ![nat](../../images/nat.PNG)
  
## Trạng thái của các loại Node Affinity
Tên của mỗi loại mô tả hành vi ở hai giai đoạn trong vòng đời Pod:
- **DuringScheduling**: lúc Pod chưa tồn tại và đang được tạo/lập lịch lần đầu.
  - `required`: scheduler **bắt buộc** đặt Pod lên node thỏa mãn quy tắc affinity. Không tìm được node phù hợp thì Pod không được lập lịch (`Pending`). Dùng khi vị trí đặt Pod là quan trọng.
  - `preferred`: scheduler **cố gắng** tìm node phù hợp; nếu không có, Pod vẫn được đặt lên node bất kỳ còn khả dụng. Dùng khi vị trí đặt Pod ít quan trọng hơn việc chạy được workload.
- **DuringExecution**: lúc Pod đang chạy mà môi trường thay đổi (ví dụ: ai đó xóa label của node).
  - `Ignored`: Pod đang chạy không bị ảnh hưởng bởi thay đổi đó.
  - `Required` (dự kiến): Pod đang chạy trên node không còn thỏa quy tắc sẽ bị evict.

  ![nats](../../images/nats.PNG)
  
  ![nats1](../../images/nats1.PNG)

> 💡 **Bổ sung:**
> - Tính đến các phiên bản Kubernetes hiện tại (v1.30+), hai loại `...RequiredDuringExecution` **vẫn chưa** được triển khai. Trong thực tế và khi thi chỉ dùng hai loại `...IgnoredDuringExecution`.
> - Các toán tử được hỗ trợ trong `matchExpressions`: `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt`, `Lt` (`Gt`/`Lt` so sánh giá trị label dạng số nguyên). `NotIn` và `DoesNotExist` có thể dùng để tạo hiệu ứng "node anti-affinity".
> - Logic kết hợp: nhiều phần tử trong `nodeSelectorTerms` là **OR** (chỉ cần một term thỏa), còn nhiều biểu thức trong cùng một `matchExpressions` là **AND** (phải thỏa tất cả).
> - Ví dụ `preferredDuringSchedulingIgnoredDuringExecution` — mỗi quy tắc có `weight` (1–100); scheduler cộng điểm cho node thỏa quy tắc và ưu tiên node có tổng điểm cao hơn:
>   ```
>   affinity:
>     nodeAffinity:
>       preferredDuringSchedulingIgnoredDuringExecution:
>       - weight: 1
>         preference:
>           matchExpressions:
>           - key: size
>             operator: In
>             values:
>             - Large
>   ```
> - Mẹo thi: không cần nhớ thuộc lòng cú pháp. Trên trang https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/ có sẵn YAML mẫu để copy; hoặc dùng `kubectl explain pod.spec.affinity.nodeAffinity --recursive`.
> - Ngoài node affinity còn có **Pod affinity / Pod anti-affinity** (`podAffinity`, `podAntiAffinity`) — đặt Pod gần/xa các Pod khác dựa trên label của Pod và `topologyKey` (ví dụ `kubernetes.io/hostname` để trải các replica ra nhiều node khác nhau).
  
#### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/
- https://kubernetes.io/blog/2017/03/advanced-scheduling-in-kubernetes/
