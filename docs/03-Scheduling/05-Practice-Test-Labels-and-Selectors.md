# Practice Test - Labels và Selectors (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-labels-and-selectors/)

Lời giải cho bài Practice Test - Labels và Selectors
1.  <details>
    <summary>Chúng tôi đã triển khai một số Pod, được gắn các label tier, env và bu. Có bao nhiêu Pod thuộc môi trường dev (env)?</summary>

    Ở đây ta lọc Pod theo giá trị của một label

    ```
    kubectl get pods --selector env=dev
    ```

    Đếm số Pod (nếu có). Mẹo: thêm `--no-headers | wc -l` để đếm nhanh.

    </details>

1.  <details>
    <summary>Có bao nhiêu Pod thuộc business unit (bu) finance?</summary>

    Tương tự ...

    ```
    kubectl get pods --selector bu=finance
    ```

    Đếm số Pod (nếu có)
    </details>

1.  <details>
    <summary>Có bao nhiêu đối tượng trong môi trường prod, bao gồm Pod, ReplicaSet và mọi đối tượng khác?</summary>

    ```
    kubectl get all --selector env=prod
    ```

    Đếm tất cả (nếu có). Lưu ý `kubectl get all` liệt kê Pod, Service, Deployment, ReplicaSet, DaemonSet, StatefulSet, Job, CronJob... nhưng không bao gồm ConfigMap, Secret, v.v.
    </details>

1.  <details>
    <summary>Xác định Pod thuộc môi trường prod, BU finance và tier frontend?</summary>

    Ta có thể kết hợp nhiều biểu thức label bằng dấu phẩy. Chỉ những đối tượng có _tất cả_ các cặp label/value đã cho mới được trả về, tức là điều kiện `and`.

    ```
    kubectl get all --selector env=prod,bu=finance,tier=frontend
    ```
    </details>

1. <details>
   <summary>Cho sẵn file định nghĩa ReplicaSet replicaset-definition-1.yaml. Hãy thử tạo ReplicaSet. File này có lỗi, hãy sửa nó.</summary>

   ```
   kubectl create -f replicaset-definition-1.yaml
   ```

   Để ý thông báo lỗi (dạng `selector does not match template labels`).

   `selector.matchLabels` phải khớp với label của Pod (trong `template.metadata.labels`) - Sửa file `replicaset-definition-2.yaml`

   Giá trị label ở dòng 9 và dòng 13 phải giống nhau.

   ```
   $ kubectl create -f replicaset-definition-2.yaml
   ```
   </details>

