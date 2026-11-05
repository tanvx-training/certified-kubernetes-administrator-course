# Multi-container Pods - Pod nhiều container (10%)

> 💡 **Bổ sung:** Các container trong cùng một Pod dùng chung network namespace (gọi nhau qua `localhost`, không được trùng port) và có thể dùng chung volume. Các mẫu thiết kế phổ biến: **sidecar** (ví dụ container đẩy log), **adapter** (chuẩn hóa output), **ambassador** (proxy ra bên ngoài), cùng với **init container** chạy tuần tự đến khi xong trước khi các container chính khởi động.

### Tạo một Pod có hai container, cả hai dùng image busybox và chạy lệnh "echo hello; sleep 3600". Kết nối vào container thứ hai và chạy 'ls'

<details><summary>Xem lời giải</summary>
<p>

Cách dễ nhất là tạo một pod có một container rồi lưu định nghĩa của nó vào file YAML:

```bash
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run=client -- /bin/sh -c 'echo hello;sleep 3600' > pod.yaml
vi pod.yaml
```

Copy/paste phần thông tin của container, để YAML cuối cùng chứa hai container như dưới đây (nhớ đặt tên hai container khác nhau):

```YAML
containers:
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: busybox
    name: busybox2
```

```bash
kubectl create -f pod.yaml
# Kết nối vào container busybox2 trong pod
kubectl exec -it busybox -c busybox2 -- /bin/sh
ls
exit

# hoặc làm việc trên chỉ với một dòng lệnh
kubectl exec -it busybox -c busybox2 -- ls

# dọn dẹp
kubectl delete po busybox
```

> 💡 **Bổ sung:** Với Pod nhiều container, nếu không chỉ định `-c`, `kubectl exec` và `kubectl logs` sẽ dùng container mặc định (container đầu tiên, hoặc container được chỉ định qua annotation `kubectl.kubernetes.io/default-container`) và in ra thông báo nhắc bạn. Xem log của tất cả container: `kubectl logs busybox --all-containers`.

</p>
</details>

### Tạo một pod có container nginx mở port 80. Thêm một init container busybox tải một trang bằng lệnh 'echo "Test" > /work-dir/index.html'. Tạo một volume kiểu emptyDir và mount nó vào cả hai container. Với container nginx, mount vào "/usr/share/nginx/html", còn với init container, mount vào "/work-dir". Làm xong thì lấy IP của pod vừa tạo, tạo một pod busybox và chạy "wget -O- IP"

<details><summary>Xem lời giải</summary>
<p>

Cách dễ nhất là tạo một pod có một container rồi lưu định nghĩa của nó vào file YAML:

```bash
kubectl run box --image=nginx --restart=Never --port=80 --dry-run=client -o yaml > pod-init.yaml
```

Copy/paste phần thông tin của container, để YAML cuối cùng chứa volume và initContainer:

Volume:

```YAML
containers:
- image: nginx
...
  volumeMounts:
  - name: vol
    mountPath: /usr/share/nginx/html
volumes:
- name: vol
  emptyDir: {}
```

initContainer:

```YAML
...
initContainers:
- args:
  - /bin/sh
  - -c
  - echo "Test" > /work-dir/index.html
  image: busybox
  name: box
  volumeMounts:
  - name: vol
    mountPath: /work-dir
```

Tổng hợp lại, ta có:

```YAML

apiVersion: v1
kind: Pod
metadata:
  labels:
    run: box
  name: box
spec:
  initContainers: 
  - args: 
    - /bin/sh 
    - -c 
    - echo "Test" > /work-dir/index.html
    image: busybox 
    name: box 
    volumeMounts: 
    - name: vol 
      mountPath: /work-dir 
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    volumeMounts: 
    - name: vol 
      mountPath: /usr/share/nginx/html 
  volumes: 
  - name: vol 
    emptyDir: {} 
```

```bash
# Apply pod
kubectl apply -f pod-init.yaml

# Lấy IP
kubectl get po -o wide

# Chạy wget
kubectl run box-test --image=busybox --restart=Never -it --rm -- /bin/sh -c "wget -O- $(kubectl get pod box -o jsonpath='{.status.podIP}')"

# dọn dẹp
kubectl delete po box
```

> 💡 **Bổ sung:**
> - Trong lúc init container chạy, `kubectl get po` hiển thị trạng thái `Init:0/1`; nếu init container lỗi sẽ thấy `Init:Error` hoặc `Init:CrashLoopBackOff`. Xem log của init container bằng `kubectl logs box -c box`.
> - `emptyDir` được tạo khi Pod được gán vào node và bị xóa khi Pod bị xóa khỏi node, nên phù hợp để chia sẻ dữ liệu tạm giữa các container trong cùng Pod.
> - **Native sidecar container (v1.29 beta, GA ở v1.33):** khai báo một container trong `initContainers` với `restartPolicy: Always`. Container này khởi động trước các container chính, chạy suốt vòng đời Pod (các init container tiếp theo không phải đợi nó chạy xong) và chỉ dừng sau khi các container chính đã dừng. Đây là cách khuyến nghị hiện nay để chạy sidecar (ví dụ log shipper), nên nắm vững cho đề CKA/CKAD theo curriculum mới.

</p>
</details>

