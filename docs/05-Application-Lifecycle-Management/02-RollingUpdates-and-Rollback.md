# Rolling Updates và Rollback
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/rolling-updates-and-rollbacks/)
  
Trong phần này, chúng ta sẽ tìm hiểu về rolling update và rollback trong một Deployment.

## Rollout và quản lý phiên bản (versioning) trong Deployment

  ![rollv](../../images/rollv.PNG)

Khi bạn tạo một Deployment lần đầu, nó sẽ kích hoạt một **rollout**. Mỗi rollout tạo ra một **revision** (phiên bản) mới của Deployment. Ví dụ, lần đầu tạo Deployment ta có Revision 1; khi nâng cấp image của ứng dụng, một rollout mới được kích hoạt và tạo ra Revision 2. Nhờ vậy ta có thể theo dõi các thay đổi và quay lại (rollback) phiên bản trước nếu cần.
  
## Các lệnh rollout
- Xem trạng thái của rollout bằng lệnh sau
  ```
  $ kubectl rollout status deployment/myapp-deployment
  ```
- Xem lịch sử và các revision
  ```
  $ kubectl rollout history deployment/myapp-deployment
  ```
 
  ![rollc](../../images/rollc.PNG)

> 💡 **Bổ sung:**
> - Xem chi tiết một revision cụ thể: `kubectl rollout history deployment/myapp-deployment --revision=2`.
> - Cột `CHANGE-CAUSE` trong output của `rollout history` lấy từ annotation `kubernetes.io/change-cause`. Flag `--record` dùng để ghi annotation này đã bị deprecated; thay vào đó hãy tự đặt annotation: `kubectl annotate deployment/myapp-deployment kubernetes.io/change-cause="update image to nginx:1.9.1"`.
> - Có thể tạm dừng/tiếp tục rollout bằng `kubectl rollout pause` / `kubectl rollout resume`, và khởi động lại toàn bộ Pod (vd sau khi đổi ConfigMap) bằng `kubectl rollout restart deployment/myapp-deployment`.
  
## Các chiến lược triển khai (Deployment Strategies)
- Có 2 loại deployment strategy
  1. Recreate
  2. RollingUpdate (chiến lược mặc định)
  
  ![dst](../../images/dst.PNG)

- **Recreate**: xóa toàn bộ Pod của phiên bản cũ rồi mới tạo các Pod phiên bản mới. Cách này khiến ứng dụng bị gián đoạn (downtime) trong khoảng thời gian giữa hai bước.
- **RollingUpdate**: lần lượt thay thế từng nhóm nhỏ Pod cũ bằng Pod mới, nên ứng dụng không bị gián đoạn.

> 💡 **Bổ sung:** Với RollingUpdate, có thể điều chỉnh tốc độ thay thế qua hai trường trong `spec.strategy.rollingUpdate`:
> - `maxUnavailable`: số Pod tối đa được phép không sẵn sàng trong quá trình update (số tuyệt đối hoặc %; mặc định `25%`).
> - `maxSurge`: số Pod tối đa được phép tạo vượt quá số `replicas` mong muốn (mặc định `25%`).
>
> Khi đổi sang `type: Recreate`, phải xóa khối `rollingUpdate` đi, nếu không API server sẽ báo lỗi validation.
  
## kubectl apply
- Để cập nhật một Deployment, hãy sửa file định nghĩa Deployment với các thay đổi cần thiết và lưu lại. Sau đó chạy lệnh bên dưới.
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
   name: myapp-deployment
   labels:
    app: nginx
  spec:
   template:
     metadata:
       name: myap-pod
       labels:
         app: myapp
         type: front-end
     spec:
      containers:
      - name: nginx-container
        image: nginx:1.7.1
   replicas: 3
   selector:
    matchLabels:
      type: front-end       
  ```
  ```
  $ kubectl apply -f deployment-definition.yaml
  ```
- Một cách khác để cập nhật Deployment, ví dụ khi muốn cập nhật image.
  ```
  $ kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
  ```
  ![ka](../../images/ka.PNG)

> 💡 **Lưu ý:** Trong `kubectl set image deployment/<tên> <tên-container>=<image>`, phần bên trái dấu `=` là **tên container** chứ không phải tên image. Với YAML ở trên, lệnh chính xác là `kubectl set image deployment/myapp-deployment nginx-container=nginx:1.9.1`. Ngoài ra, dùng `kubectl set image` thì file YAML trên máy sẽ không còn khớp với cấu hình thực tế trong cluster — hãy nhớ cập nhật lại file nếu bạn quản lý cấu hình bằng file.
  
## Recreate và RollingUpdate
  
  ![rcrl](../../images/rcrl.PNG)

Có thể thấy sự khác biệt khi xem `kubectl describe deployment`: với Recreate, các event cho thấy ReplicaSet cũ bị scale xuống 0 trước, sau đó ReplicaSet mới được scale lên 5. Với RollingUpdate, ReplicaSet cũ được scale xuống từng bước, xen kẽ với việc scale ReplicaSet mới lên.
  
## Nâng cấp (Upgrades)

  ![up](../../images/up.PNG)

Khi nâng cấp ứng dụng, Deployment tạo một ReplicaSet mới và bắt đầu triển khai các Pod mới vào đó, đồng thời gỡ dần các Pod trong ReplicaSet cũ theo chiến lược rolling update. Dùng `kubectl get replicasets` để thấy ReplicaSet cũ có 0 Pod và ReplicaSet mới có đủ số Pod.
  
## Rollback
  
  ![rb](../../images/rb.PNG)
  
- Để hoàn tác (undo) một thay đổi
  ```
  $ kubectl rollout undo deployment/myapp-deployment
  ```

Deployment sẽ hủy các Pod trong ReplicaSet mới và khôi phục các Pod cũ trong ReplicaSet cũ.

> 💡 **Bổ sung:** Rollback về một revision cụ thể bằng `kubectl rollout undo deployment/myapp-deployment --to-revision=1`. Lưu ý sau khi rollback, revision cũ được "đánh số lại" thành revision mới nhất trong lịch sử. Số revision được giữ lại do `spec.revisionHistoryLimit` quyết định (mặc định 10).
  
## kubectl create
- Để tạo một Deployment
  ```
  $ kubectl create deployment nginx --image=nginx
  ```

> 💡 **Mẹo thi:** Tạo nhanh file YAML mẫu rồi chỉnh sửa: `kubectl create deployment nginx --image=nginx --replicas=3 --dry-run=client -o yaml > deploy.yaml`.

## Tổng hợp các lệnh kubectl
```
$ kubectl create -f deployment-definition.yaml
$ kubectl get deployments
$ kubectl apply -f deployment-definition.yaml
$ kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
$ kubectl rollout status deployment/myapp-deployment
$ kubectl rollout history deployment/myapp-deployment
$ kubectl rollout undo deployment/myapp-deployment
```

![sum](../../images/sum.PNG)
 
#### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/concepts/workloads/controllers/deployment
- https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment
