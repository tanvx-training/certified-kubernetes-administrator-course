# Practice Test - Env Variables (Bài thực hành biến môi trường)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-env-variables/)
  
Lời giải cho bài practice test env variables
- Chạy lệnh 'kubectl get pods' và đếm số lượng Pod.
  
  <details>
  
  ```
  $ kubectl get pods
  ```
  
  </details>
  
- Chạy lệnh 'kubectl describe pod' và tìm phần ENV (Environment).

  <details>
  
  ```
  $ kubectl describe pod
  ```
  
  </details>
  
- Chạy lệnh 'kubectl describe pod' và tìm phần ENV (Environment).
  
  <details>
  
  ```
  $ kubectl describe pod
  ```
  
  </details>
    
- Xem giao diện web của ứng dụng bằng cách nhấn vào tab 'Webapp Color' phía trên terminal.

- Đặt biến môi trường APP_COLOR thành green.
  
  <details>
  
  ```
  $ kubectl get pods webapp-color -o yaml > green.yaml
  $ kubectl delete pods webapp-color
  Cập nhật APP_COLOR thành green
  $ kubectl create -f green.yaml
  ```

  > 💡 **Giải thích:** Không thể sửa `env` của một Pod đang chạy, nên phải xuất YAML, xóa Pod, sửa file rồi tạo lại. Cách nhanh hơn: `kubectl edit pod webapp-color` → sửa → lưu (bị từ chối, bản sửa được lưu ở `/tmp/`) → `kubectl replace --force -f /tmp/kubectl-edit-xxxx.yaml`.
  
  </details>
  
- Xem thay đổi trên giao diện web bằng cách nhấn vào tab 'Webapp Color' phía trên terminal.

- Chạy lệnh kubectl get configmaps
  
  <details>
  
  ```
  $ kubectl get configmaps
  ```
  
  </details>
  
- Chạy lệnh 'kubectl describe configmaps' và tìm key DB_HOST.

  <details>
  
  ```
  $ kubectl describe configmaps
  ```
  
  </details>
  
- Tạo một ConfigMap mới cho Pod 'webapp-color'. Dùng spec được cho ở bên phải.

  <details>
  
  ```
  $ kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue
  ```

  > 💡 **Lưu ý:** Đề bài có thể yêu cầu nhiều key (vd thêm `APP_OTHER=disregard`); khi đó lặp lại `--from-literal` cho từng key. Kiểm tra lại bằng `kubectl describe configmap webapp-config-map`.
  
  </details>
  
- Đặt biến môi trường bằng envFrom và dùng configMapRef webapp-config-map.
  
  <details>
  
  ```
  $ kubectl get pods webapp-color -o yaml > new-webapp.yaml
  $ kubectl delete pods webapp-color
   Cập nhật file định nghĩa Pod, trong phần spec.containers thêm nội dung dưới đây.
  - envFrom:
    - configMapRef:
       name: webapp-config-map
  ```
  
  </details>
  
  <details>
  
  ```
  $ kubectl create -f new-webapp.yaml
  ``` 
  
  </details>

  > 💡 **Lưu ý:** `envFrom` là một trường của container (cùng cấp với `name`, `image`), không phải một phần tử mới của danh sách `containers`. Nếu container đang có `env` đặt `APP_COLOR`, hãy xóa nó đi — khi một biến được khai báo ở cả `env` và `envFrom`, giá trị trong `env` sẽ được ưu tiên.

- Xem thay đổi trên giao diện web bằng cách nhấn vào tab 'Webapp Color' phía trên terminal.
