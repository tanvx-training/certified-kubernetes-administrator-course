# Labels và Selectors
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/labels-and-selectors/)
  
Trong phần này, chúng ta sẽ tìm hiểu về **`Labels và Selectors`**

#### Labels và Selectors là cách tiêu chuẩn để nhóm các đối tượng lại với nhau.
  
#### Labels là các thuộc tính (cặp key/value) gắn vào từng đối tượng.

  ![labels-ckc](../../images/labels-ckc.PNG)
  
#### Selectors giúp bạn lọc các đối tượng đó theo label.
 
  ![sl](../../images/sl.PNG)
  
Labels và selectors được dùng trong Kubernetes như thế nào?
- Chúng ta đã tạo rất nhiều loại đối tượng khác nhau trong Kubernetes như **`Pod`**, **`ReplicaSet`**, **`Deployment`**, v.v. Khi số lượng tăng lên hàng trăm, hàng nghìn, ta cần cách để nhóm và lọc chúng theo loại, theo ứng dụng, theo chức năng...
  
  ![ls](../../images/ls.PNG)
  
Khai báo label như thế nào?
   ```
    apiVersion: v1
    kind: Pod
    metadata:
     name: simple-webapp
     labels:
       app: App1
       function: Front-end
    spec:
     containers:
     - name: simple-webapp
       image: simple-webapp
       ports:
       - containerPort: 8080
   ```
 ![lpod](../../images/lpod.PNG)
 
Sau khi Pod được tạo, để chọn Pod theo label, chạy lệnh sau
```
$ kubectl get pods --selector app=App1
```

Kubernetes dùng label để liên kết các đối tượng khác nhau với nhau. Ví dụ, ReplicaSet dùng `selector` để tìm các Pod mà nó quản lý. Lưu ý có hai chỗ khai báo label: `metadata.labels` ở đầu file là label của chính ReplicaSet, còn `template.metadata.labels` là label của Pod. `selector.matchLabels` phải khớp với label của Pod trong template.
   ```
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: simple-webapp
      labels:
        app: App1
        function: Front-end
    spec:
     replicas: 3
     selector:
       matchLabels:
        app: App1
     template:
       metadata:
         labels:
           app: App1
           function: Front-end
       spec:
         containers:
         - name: simple-webapp
           image: simple-webapp   
   ```

  ![lrs](../../images/lrs.PNG)

Với Service, `selector` dùng để xác định các Pod mà Service sẽ chuyển traffic tới
 
      ```
      apiVersion: v1
      kind: Service
      metadata:
       name: my-service
      spec:
       selector:
         app: App1
       ports:
       - protocol: TCP
         port: 80
         targetPort: 9376 
       ```
  ![lrs1](../../images/lrs1.PNG)
  
## Annotations
- Trong khi labels và selectors dùng để nhóm các đối tượng, annotations dùng để ghi lại các thông tin khác mang tính tham khảo (ví dụ: phiên bản build, tên người phụ trách, số điện thoại liên hệ...). Annotation không dùng để lọc/chọn đối tượng.
    ```
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: simple-webapp
      labels:
        app: App1
        function: Front-end
      annotations:
         buildversion: 1.34
    spec:
     replicas: 3
     selector:
       matchLabels:
        app: App1
    template:
      metadata:
        labels:
          app: App1
          function: Front-end
      spec:
        containers:
        - name: simple-webapp
          image: simple-webapp   
    ```
  ![annotations](../../images/annotations.PNG)

> 💡 **Bổ sung — các thao tác label hay dùng khi thi:**
> - Lọc nhiều label (điều kiện AND): `kubectl get pods -l env=prod,tier=frontend` (`-l` là dạng viết tắt của `--selector`).
> - Selector dạng tập hợp (set-based): `kubectl get pods -l 'env in (prod,dev)'`, `kubectl get pods -l '!tier'`, `kubectl get pods -l 'env!=prod'`.
> - Hiển thị label: `kubectl get pods --show-labels`, hoặc hiển thị một label thành cột riêng: `kubectl get pods -L env`.
> - Thêm/sửa/xóa label: `kubectl label pod mypod env=dev`, `kubectl label pod mypod env=prod --overwrite`, `kubectl label pod mypod env-`.
> - Thêm annotation: `kubectl annotate pod mypod buildversion=1.34`.
> - Đếm nhanh số đối tượng: `kubectl get pods -l env=dev --no-headers | wc -l`.
> - Trong `Deployment`/`ReplicaSet` (`apps/v1`), `spec.selector` là bắt buộc và không thể thay đổi sau khi tạo. Ngoài `matchLabels`, còn có `matchExpressions` (toán tử `In`, `NotIn`, `Exists`, `DoesNotExist`).
> - Lưu ý: trong YAML trên, giá trị annotation `1.34` sẽ được hiểu là số; thực tế giá trị annotation/label phải là chuỗi, nên nên viết `buildversion: "1.34"` để tránh lỗi khi apply.

Tài liệu tham khảo K8s:
- https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
