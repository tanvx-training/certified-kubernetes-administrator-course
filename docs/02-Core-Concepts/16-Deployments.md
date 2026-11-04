# Deployments
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/deployments-3/)

Trong phần này, chúng ta sẽ tìm hiểu về Deployment trong Kubernetes.

#### Deployment là một object của Kubernetes.
  
 ![deployment](../../images/deployment.PNG)

> 💡 **Bổ sung:** Deployment nằm ở tầng cao hơn ReplicaSet trong hệ thống phân cấp: **Deployment → ReplicaSet → Pod**. Ngoài khả năng duy trì số replica (do ReplicaSet đảm nhiệm), Deployment còn cung cấp:
> - **Rolling update:** cập nhật phiên bản ứng dụng (ví dụ đổi image) lần lượt từng Pod, không gây downtime.
> - **Rollback:** quay về phiên bản trước nếu bản cập nhật gặp lỗi (`kubectl rollout undo`).
> - **Pause/Resume:** tạm dừng rollout để gom nhiều thay đổi rồi áp dụng một lần.
>
> Các tính năng này sẽ được trình bày chi tiết ở phần Application Lifecycle Management.
  
#### Tạo Deployment như thế nào?

```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: myapp-deployment
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
- Khi file đã sẵn sàng, tạo Deployment bằng file định nghĩa Deployment
  ```
  $ kubectl create -f deployment-definition.yaml
  ```
- Xem Deployment vừa tạo
  ```
  $ kubectl get deployment
  ```
- Deployment tự động tạo một **`ReplicaSet`**. Để xem các ReplicaSet
  ```
  $ kubectl get replicaset
  ```
- Cuối cùng, ReplicaSet tạo ra các **`Pod`**. Để xem các Pod
  ```
  $ kubectl get pods
  ```
    
  ![deployment1](../../images/deployment1.PNG)
  
- Xem tất cả các object cùng lúc
  ```
  $ kubectl get all
  ```
  ![deployment2](../../images/deployment2.PNG)

> 💡 **Bổ sung:** Để ý cách đặt tên: ReplicaSet có tên dạng `<tên-deployment>-<pod-template-hash>` (ví dụ `myapp-deployment-6795844b58`), còn Pod có tên `<tên-replicaset>-<chuỗi ngẫu nhiên>`. Trong kỳ thi, thay vì viết YAML từ đầu, hãy sinh nhanh bằng lệnh imperative: `kubectl create deployment myapp-deployment --image=nginx --replicas=3 --dry-run=client -o yaml > deployment-definition.yaml`.
>
> Lưu ý `kubectl get all` không hiển thị mọi loại resource (ví dụ không có ConfigMap, Secret, Ingress, PVC...).
  
Tài liệu tham khảo K8s:
- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
- https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/
- https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/
- https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/
