# Cấu hình ConfigMap cho ứng dụng (Configure ConfigMaps in Applications)
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/configure-configmaps-in-applications/)
  
Trong phần này, chúng ta sẽ tìm hiểu cách cấu hình ConfigMap cho ứng dụng.

Khi có nhiều file định nghĩa Pod, việc quản lý biến môi trường trực tiếp trong từng file trở nên khó khăn. ConfigMap cho phép tách dữ liệu cấu hình ra khỏi file định nghĩa Pod và quản lý tập trung dưới dạng các cặp key-value.

## ConfigMaps
- Có 2 bước khi cấu hình ConfigMap:
  - Bước 1: tạo ConfigMap
  - Bước 2: inject (đưa) ConfigMap vào Pod.
- Có 2 cách để tạo ConfigMap:
  - Cách imperative (dùng lệnh)
    ```
    $ kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=prod
    $ kubectl create configmap app-config --from-file=app_config.properties (Another way)
    ```
    ![cmi](../../images/cmi.PNG)
    
  - Cách declarative (dùng file định nghĩa)
    
    ```
    apiVersion: v1
    kind: ConfigMap
    metadata:
     name: app-config
    data:
     APP_COLOR: blue
     APP_MODE: prod
    ```
    ```
    Tạo file định nghĩa ConfigMap rồi chạy lệnh `kubectl create` để triển khai.
    $ kubectl create -f config-map.yaml
    ```
    ![cmd1](../../images/cmd1.PNG)

> 💡 **Bổ sung:** Với `--from-file`, mặc định **tên file** trở thành key và **nội dung file** trở thành value (vd key `app_config.properties`). Có thể đặt key khác bằng `--from-file=<key>=<đường-dẫn>`. Nếu muốn mỗi dòng `KEY=VALUE` trong file thành một key riêng, dùng `--from-env-file=app_config.properties`. Tạo nhanh YAML mẫu: `kubectl create configmap app-config --from-literal=APP_COLOR=blue --dry-run=client -o yaml`.
    
 ## Xem ConfigMap
 - Liệt kê các ConfigMap
   ```
   $ kubectl get configmaps (or)
   $ kubectl get cm
   ```
 - Xem chi tiết ConfigMap
   ```
   $ kubectl describe configmaps
   ```
   
   ![cmv](../../images/cmv.PNG)
   
 ## Sử dụng ConfigMap trong Pod
 - Inject ConfigMap vào Pod (dùng `envFrom` để nạp toàn bộ key của ConfigMap thành biến môi trường)
   ```
   apiVersion: v1
   kind: Pod
   metadata:
     name: simple-webapp-color
   spec:
    containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
      - containerPort: 8080
      envFrom:
      - configMapRef:
          name: app-config
   ```
   ```
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: app-config
   data:
     APP_COLOR: blue
     APP_MODE: prod
   ```
   ```
   $ kubectl create -f pod-definition.yaml
   ```
  
   ![cmp](../../images/cmp.PNG)
   
 #### Các cách khác để inject biến cấu hình vào Pod
 - Inject dưới dạng **`một biến môi trường riêng lẻ`** (Single Environment Variable, dùng `valueFrom.configMapKeyRef`)
 - Inject dưới dạng file trong một **`Volume`**
 
   ![cmp1](../../images/cmp1.PNG)

> 💡 **Bổ sung:**
> - Khi mount ConfigMap dưới dạng volume, **mỗi key trở thành một file** trong thư mục mount, nội dung file là value. Có thể chọn một số key bằng `items`, hoặc mount một file đơn lẻ bằng `subPath`.
> - Biến môi trường lấy từ ConfigMap chỉ được đọc **khi container khởi động**; sửa ConfigMap sau đó không làm thay đổi biến trong container đang chạy — cần tạo lại Pod (vd `kubectl rollout restart deployment/<tên>`). Ngược lại, file mount từ volume sẽ được kubelet cập nhật sau một khoảng trễ (trừ khi dùng `subPath`).
> - ConfigMap phải tồn tại (và cùng namespace với Pod) trước khi Pod khởi động, nếu không Pod sẽ báo lỗi `CreateContainerConfigError` (trừ khi đánh dấu `optional: true`).
> - Có thể đặt `immutable: true` cho ConfigMap để ngăn chỉnh sửa và giảm tải cho kube-apiserver.
> - ConfigMap không mã hóa dữ liệu; với dữ liệu nhạy cảm hãy dùng Secret. Dung lượng tối đa của một ConfigMap là 1 MiB.
   
 #### Tài liệu tham khảo K8s
 - https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
 - https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#define-container-environment-variables-using-configmap-data
 - https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#create-configmaps-from-files
