# Observability - Khả năng quan sát (18%)

## Liveness, readiness and startup probes

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

> 💡 **Bổ sung:** Ba loại probe có vai trò khác nhau:
> - **livenessProbe**: kiểm tra container còn "sống" không. Thất bại quá `failureThreshold` lần thì kubelet kill container và restart theo `restartPolicy`.
> - **readinessProbe**: kiểm tra container đã sẵn sàng nhận traffic chưa. Thất bại thì Pod bị gỡ khỏi endpoint của Service (cột `READY` hiện `0/1`), nhưng container **không** bị restart.
> - **startupProbe**: dành cho ứng dụng khởi động chậm. Khi startupProbe chưa thành công, liveness và readiness probe chưa được chạy.
>
> Mỗi probe có thể dùng một trong bốn cơ chế: `exec`, `httpGet`, `tcpSocket`, và `grpc` (GA từ v1.27). Các tham số thời gian chính: `initialDelaySeconds` (mặc định 0), `periodSeconds` (mặc định 10), `timeoutSeconds` (mặc định 1), `failureThreshold` (mặc định 3), `successThreshold` (mặc định 1).

### Tạo một pod nginx có liveness probe chỉ chạy lệnh 'ls'. Lưu YAML của nó vào pod.yaml. Chạy pod, kiểm tra trạng thái probe, rồi xóa pod.

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    livenessProbe: # probe của chúng ta
      exec: # thêm dòng này
        command: # định nghĩa lệnh
        - ls # lệnh ls
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl describe pod nginx | grep -i liveness # chạy lệnh này để thấy liveness probe hoạt động
kubectl delete -f pod.yaml
```

> 💡 **Bổ sung:** Với `exec` probe, kubelet coi probe thành công khi lệnh trả về exit code 0. Lệnh `ls` luôn thành công nên probe này luôn pass. Để thấy probe thất bại, thử đổi thành `cat /tmp/healthy` (file không tồn tại) rồi quan sát event `Unhealthy` trong `kubectl describe`.

</p>
</details>

### Sửa file pod.yaml để liveness probe bắt đầu chạy sau 5 giây, và khoảng cách giữa các lần probe là 5 giây. Chạy pod, kiểm tra probe, rồi xóa pod.

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl explain pod.spec.containers.livenessProbe # lấy tên chính xác của các trường
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    livenessProbe:
      initialDelaySeconds: 5 # thêm dòng này
      periodSeconds: 5 # thêm cả dòng này nữa
      exec:
        command:
        - ls
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl describe po nginx | grep -i liveness
kubectl delete -f pod.yaml
```

> 💡 **Bổ sung:** `kubectl explain` là "tài liệu offline" rất hữu ích khi thi. Thêm `--recursive` để xem toàn bộ cây trường, ví dụ `kubectl explain pod.spec.containers.livenessProbe --recursive`.

</p>
</details>

### Tạo một pod nginx (có khai báo port 80) với readinessProbe kiểu HTTP trên path '/' port 80. Tương tự, chạy pod, kiểm tra readinessProbe, rồi xóa pod.

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml --restart=Never --port=80 > pod.yaml
vi pod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    ports:
    - containerPort: 80 # Lưu ý: readiness probe chạy trên container trong suốt vòng đời của nó. Vì nginx đã lắng nghe port 80, khai báo containerPort: 80 không bắt buộc để readiness hoạt động.
    readinessProbe: # khai báo readiness probe
      httpGet: # thêm dòng này
        path: / #
        port: 80 #
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl describe pod nginx | grep -i readiness # để xem chi tiết readiness của pod
kubectl delete -f pod.yaml
```

> 💡 **Bổ sung:** `httpGet` probe thành công khi HTTP status code nằm trong khoảng 200–399. Trường `port` có thể là số hoặc tên port (ví dụ `port: http` nếu bạn đặt `name: http` cho `containerPort`).

</p>
</details>

### Có nhiều pod đang chạy trong các namespace `qa`,`alan`,`test`,`production`. Tất cả các pod này đều được cấu hình liveness probe. Hãy liệt kê tất cả các pod có liveness probe bị thất bại theo định dạng `<namespace>/<pod name>`, mỗi pod một dòng.

<details><summary>Xem lời giải</summary>
<p>

Một event điển hình khi liveness probe thất bại:
```
LAST SEEN   TYPE      REASON      OBJECT              MESSAGE
22m         Warning   Unhealthy   pod/liveness-exec   Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
```

Thu thập các pod bị lỗi theo từng namespace

```sh
kubectl get events -o json | jq -r '.items[] | select(.message | contains("Liveness probe failed")).involvedObject | .namespace + "/" + .name'
```

> 💡 **Bổ sung:**
> - Lệnh trên chỉ lấy event của namespace hiện tại. Chạy lần lượt với `-n qa`, `-n alan`, ... hoặc thêm `-A` để lấy tất cả namespace một lần: `kubectl get events -A -o json | jq ...`. Nếu có thể có nhiều event cho cùng một pod, thêm `| sort -u` vào cuối để loại trùng lặp.
> - Cách không cần `jq`: `kubectl get events -A --field-selector reason=Unhealthy` rồi lọc các dòng chứa `Liveness probe failed`.
> - Lưu ý event mặc định chỉ được giữ khoảng 1 giờ (`--event-ttl` của kube-apiserver), nên pod lỗi đã lâu có thể không còn event. Khi đó xem thêm cột `RESTARTS` trong `kubectl get po -A`.

</p>
</details>

## Logging

### Tạo một pod busybox chạy `i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done`. Kiểm tra log của nó

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done'
kubectl logs busybox -f # theo dõi log liên tục
```

> 💡 **Bổ sung:** Một số flag hữu ích của `kubectl logs`: `--tail=20` (chỉ lấy 20 dòng cuối), `--since=5m` (log trong 5 phút gần nhất), `--timestamps` (thêm thời gian), `-l app=web` (lấy log theo label). Trong CKA hay gặp dạng đề "ghi log chứa chuỗi X vào file /opt/...": `kubectl logs <pod> | grep X > /opt/...`.

</p>
</details>

## Debugging

### Tạo một pod busybox chạy 'ls /notexist'. Xác định xem có lỗi không (dĩ nhiên là có), và xem lỗi đó. Cuối cùng, xóa pod

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl run busybox --restart=Never --image=busybox -- /bin/sh -c 'ls /notexist'
# cho thấy là có lỗi
kubectl logs busybox
kubectl describe po busybox
kubectl delete po busybox
```

</p>
</details>

### Tạo một pod busybox chạy lệnh 'notexist'. Xác định xem có lỗi không (dĩ nhiên là có), và xem lỗi đó. Cuối cùng, xóa pod một cách cưỡng bức với grace period bằng 0

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl run busybox --restart=Never --image=busybox -- notexist
kubectl logs busybox # sẽ không có gì! container chưa từng khởi động
kubectl describe po busybox # trong phần events, bạn sẽ thấy lỗi
# ngoài ra...
kubectl get events | grep -i error # bạn cũng sẽ thấy lỗi ở đây
kubectl delete po busybox --force --grace-period=0
```

> 💡 **Bổ sung:** Hai bài trên minh họa hai loại lỗi khác nhau. Ở bài trước, container khởi động được nhưng lệnh `ls` trả về lỗi, nên thông báo lỗi nằm trong `kubectl logs` và Pod ở trạng thái `Error`. Ở bài này, container runtime không tìm thấy file thực thi `notexist`, nên container không bao giờ chạy: Pod ở trạng thái `StartError` (hoặc `ContainerCannotRun`, tùy runtime) và lỗi chỉ thấy trong events. Ngoài ra, `--force --grace-period=0` xóa object ngay lập tức mà không chờ kubelet xác nhận container đã dừng; dùng được khi thi để tiết kiệm thời gian, nhưng cần cẩn thận với StatefulSet ở môi trường thật.

</p>
</details>


### Lấy mức sử dụng CPU/memory của các node (yêu cầu [metrics-server](https://github.com/kubernetes-incubator/metrics-server) đang chạy)

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl top nodes
```

> 💡 **Bổ sung:** metrics-server hiện nằm ở repo `kubernetes-sigs/metrics-server`. Với Pod, dùng `kubectl top pod -A --sort-by=memory` (hoặc `--sort-by=cpu`) để tìm Pod tốn tài nguyên nhất, và `--containers` để xem theo từng container. Đề CKA thường hỏi dạng "tìm Pod có label X tiêu tốn CPU nhiều nhất và ghi tên vào file".

</p>
</details>
