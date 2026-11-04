# Practice Test - Deployments (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-tests-deployments/)

Lời giải cho bài practice test Deployments
1. <details>
   <summary>Có bao nhiêu Pod đang tồn tại trên hệ thống?</summary>

   ```bash
   kubectl get pods
   ```

   Đếm số Pod hiển thị (nếu có).

   </details>

1. <details>
   <summary>Có bao nhiêu ReplicaSet đang tồn tại trên hệ thống?</summary>

   ```bash
   kubectl get replicasets
   ```

   Đếm số ReplicaSet hiển thị (nếu có).

   </details>

1. <details>
   <summary>Có bao nhiêu Deployment đang tồn tại trên hệ thống?</summary>

   ```bash
   kubectl get deployments
   ```

   Đếm số Deployment hiển thị (nếu có).

   </details>

1. <details>
   <summary>Bây giờ có bao nhiêu Deployment trên hệ thống?</summary>

   ```bash
   kubectl get deployments
   ```

   Đếm số Deployment hiển thị (nếu có).

   </details>

1. <details>
   <summary>Bây giờ có bao nhiêu ReplicaSet trên hệ thống?</summary>

   ```bash
   kubectl get replicasets
   ```

   Đếm số ReplicaSet hiển thị (nếu có).

   </details>

1. <details>
   <summary>Có bao nhiêu Pod trên hệ thống?</summary>

   ```bash
   kubectl get pods
   ```

   Đếm số Pod hiển thị (nếu có).

   </details>

1. <details>
   <summary>Trong số các Pod hiện có, bao nhiêu Pod đang ready?</summary>

   Từ output của lệnh trước, xem cột `READY`.
   </details>

1. <details>
   <summary>Image nào được dùng để tạo các Pod trong Deployment mới?</summary>

   ```
   kubectl describe deployment
   ```

   Xem trong mục containers.

   Một cách khác - chạy lệnh sau và xem cột `IMAGES`

   ```
   kubectl get deployment -o wide
   ```
   </details>

1. <details>
   <summary>Theo bạn, vì sao Deployment không ready?</summary>

   ```
   kubectl describe pods
   ```

   Xem trong mục events.
   </details>

1. <details>
   <summary>Tạo một Deployment mới bằng file deployment-definition-1.yaml nằm ở /root/.</br>File này có lỗi, hãy thử sửa nó.</summary>

   ```
   kubectl create -f deployment-definition-1.yaml
   ```

   Để ý thông báo lỗi.

   Sửa file bằng `vi`...

   Giá trị của **`kind`** bị sai. Nó phải là **`Deployment`** với chữ **`D`** viết hoa. Sửa file định nghĩa Deployment rồi tạo Deployment bằng lệnh ở trên.

   > 💡 **Bổ sung:** Giá trị `kind` (và cả tên trường trong YAML) phân biệt chữ hoa/thường. Lỗi thường gặp có dạng `no matches for kind "deployment" in version "apps/v1"`.

   </details>

1. <details>
   <summary>Tạo một Deployment mới với các thuộc tính dưới đây, sử dụng file định nghĩa Deployment do bạn tự viết.</summary>

   Tạo file định nghĩa Deployment trong `vi`, ví dụ `my-deployment.yaml`, với nội dung sau

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: httpd-frontend
      name: httpd-frontend
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: httpd-frontend
      template:
        metadata:
          labels:
            app: httpd-frontend
        spec:
          containers:
          - image: httpd:2.4-alpine
            name: httpd
    ```

   ```
   kubectl create -f my-deployment.yaml
   ```

   Hoặc có thể tạo theo cách imperative...

   ```
   kubectl create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3
   ```

   > 💡 **Mẹo thi:** Kết hợp hai cách: dùng lệnh imperative kèm `--dry-run=client -o yaml > my-deployment.yaml` để sinh file YAML chuẩn, chỉnh sửa thêm nếu cần, rồi `kubectl create -f my-deployment.yaml`. Cách này nhanh và tránh lỗi thụt lề.

   </details>

