# Cluster 1, Storage, olive-pvc-cka10-str

Với câu hỏi này, hãy chuyển context sang `cluster1` bằng lệnh:


```
kubectl config use-context cluster1
```

Chúng ta muốn deploy một ứng dụng viết bằng Python lên cluster, sử dụng file template nằm tại `/root/olive-app-cka10-str.yaml` trên `student-node`. Tuy nhiên, trước khi tiến hành, cần sửa file YAML theo các yêu cầu dưới đây:


* File YAML cũng phải chứa một persistent volume claim tên là `olive-pvc-cka10-str` để claim `100Mi` storage từ PV `olive-pv-cka10-str`.
* Cập nhật Deployment để thêm một sidecar container dùng image `busybox` (có thể bạn cần thêm lệnh sleep để container này tiếp tục chạy).
* Chia sẻ volume `python-data` cho container này và mount nó tại đường dẫn `/usr/src`. Đảm bảo container này chỉ có quyền `read` (chỉ đọc) trên volume.
* Cuối cùng, tạo Pod từ file YAML này và đảm bảo Pod ở trạng thái `Running`.

### Đề bài không ghi, nhưng bắt buộc phải làm để qua câu này

* Tạo một Service kiểu NodePort cho Deployment này với thông số sau
    * Node port: `32006`
    * Tên: `olive-svc-cka10-str`


---

### Lời giải

1.  Xem xét những gì đang có...

    ```
    cat /root/olive-app-cka10-str
    ```

    Phần volume claim tham chiếu tới PVC đã có sẵn trong file. Kiểm tra xem PVC đã tồn tại chưa

    ```
    kubectl get pvc
    ```

    PVC chưa tồn tại, vì vậy phải tạo nó trước.

    ```
    kubectl get pv
    ```

    PV đã tồn tại. Ghi lại `ACCESS MODES` và `STORAGECLASS` – đây là các giá trị bắt buộc trong manifest của PVC, cùng với dung lượng yêu cầu được cho trong đề.

1.  Chuẩn bị manifest cho PVC mới

    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: olive-pvc-cka10-str
    spec:
      accessModes:
      - ReadWriteMany
      resources:
        requests:
          storage: 100Mi
      storageClassName: olive-stc-cka10-str
    ```

    Sau đó tạo nó. PVC sẽ chưa bind cho tới khi Pod được tạo.

    > 💡 **Bổ sung:** PVC ở trạng thái `Pending` lúc này là bình thường nếu StorageClass có `volumeBindingMode: WaitForFirstConsumer` – PVC chỉ bind khi có Pod đầu tiên sử dụng nó được schedule. Kiểm tra bằng `kubectl get sc olive-stc-cka10-str` (cột `VOLUMEBINDINGMODE`) và `kubectl describe pvc olive-pvc-cka10-str` (event `WaitForFirstConsumer`).

1.  Sửa Pod theo yêu cầu và thêm Service vào cuối file.

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: olive-app-cka10-str
    spec:
      replicas: 1
      template:
        metadata:
          labels:
            app: olive-app-cka10-str
        spec:
          affinity:
            nodeAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                - matchExpressions:
                  - key: kubernetes.io/hostname
                    operator: In
                    values:
                      - cluster1-node01
          containers:
          - name: busybox
            image: busybox
            command:        # <- Dạng lệnh sleep nào cũng được.
            - bin/sh        # Cần sleep đủ lâu cho tới khi kết thúc bài thi.
            - -c
            - sleep 10000
            volumeMounts:
            - mountPath: /usr/src
              name: python-data
              readOnly: true
          - name: python
            image: poroko/flask-demo-app
            ports:
            - containerPort: 5000
            volumeMounts:
            - name: python-data
              mountPath: /usr/share/
          volumes:
          - name: python-data
            persistentVolumeClaim:
              claimName: olive-pvc-cka10-str
      selector:
        matchLabels:
          app: olive-app-cka10-str

    ---

    apiVersion: v1
    kind: Service
    metadata:
      name: olive-svc-cka10-str
      namespace: default
    spec:
      ports:
      - nodePort: 32006
        port: 5000
        protocol: TCP
        targetPort: 5000
      selector:
        app: olive-app-cka10-str
      type: NodePort
    ```

1.  Tạo các resource

    ```
    kubectl apply -f /root/olive-app-cka10-str
    ```

> 💡 **Bổ sung:**
> - Đường dẫn file trong đề là `/root/olive-app-cka10-str.yaml`; nếu lệnh `cat`/`kubectl apply` ở trên báo không tìm thấy file thì thêm đuôi `.yaml`.
> - `readOnly: true` trong `volumeMounts` là cách đáp ứng yêu cầu "chỉ có quyền read" – đặt ở mount của sidecar, không đặt ở volume (vì container python vẫn cần ghi).
> - Trong `command`, nên viết đầy đủ `/bin/sh`; `bin/sh` vẫn chạy được ở đây chỉ vì thư mục làm việc mặc định của image busybox là `/`.
> - Kiểm tra kết quả: `kubectl get pvc olive-pvc-cka10-str` phải là `Bound`, `kubectl get pod -l app=olive-app-cka10-str` phải là `2/2 Running`, và `kubectl get svc olive-svc-cka10-str` hiển thị `5000:32006/TCP`.
> - Từ v1.29, Kubernetes còn hỗ trợ native sidecar (khai báo trong `initContainers` với `restartPolicy: Always`), nhưng với câu hỏi này, thêm một container thường vào `containers` là đủ.