# Core Concepts - Khái niệm cốt lõi (13%)

kubernetes.io > Documentation > Reference > kubectl CLI > [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

kubernetes.io > Documentation > Tasks > Monitoring, Logging, and Debugging > [Get a Shell to a Running Container](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/)

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Configure Access to Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Accessing Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/) using API

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Use Port Forwarding to Access Applications in a Cluster](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)

> 💡 **Bổ sung:** Trước khi làm bài (và ngay đầu giờ thi CKA/CKAD), nên thiết lập alias và biến môi trường để gõ nhanh hơn:
> - `alias k=kubectl`
> - `export do="--dry-run=client -o yaml"`, dùng như `k run nginx --image=nginx $do > pod.yaml`
> - `export now="--force --grace-period=0"`, dùng như `k delete po nginx $now`
>
> Môi trường thi hiện tại đã cấu hình sẵn `alias k=kubectl` và bash completion, nhưng hai biến `do`/`now` thì bạn phải tự tạo.

### Tạo một namespace tên 'mynamespace' và một pod tên nginx dùng image nginx trong namespace này

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create namespace mynamespace
kubectl run nginx --image=nginx --restart=Never -n mynamespace
```

> 💡 **Bổ sung:** Từ kubectl v1.18, `kubectl run` chỉ tạo Pod (các flag `--generator` cũ để tạo Deployment/Job đã bị loại bỏ). Vì vậy `--restart=Never` ở đây không bắt buộc để ra Pod; nó chỉ đặt `restartPolicy: Never` (mặc định là `Always`). Muốn tạo Deployment thì dùng `kubectl create deployment`.

</p>
</details>

### Tạo pod vừa mô tả ở trên bằng YAML

<details><summary>Xem lời giải</summary>
<p>

Sinh YAML nhanh bằng lệnh:

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -n mynamespace -o yaml > pod.yaml
```

```bash
cat pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
  namespace: mynamespace
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
```

Hoặc bạn có thể làm trong một dòng:

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml | kubectl create -n mynamespace -f -
```

> 💡 **Bổ sung:** `--dry-run=client` chỉ sinh object ở phía kubectl mà không gửi lên API server; `--dry-run=server` thì gửi lên để API server kiểm tra (admission, validation) nhưng không lưu. Dạng cũ `--dry-run` (không có giá trị) đã deprecated, nên luôn ghi rõ `=client`. Đây là kỹ thuật tạo YAML nhanh nhất trong cả kỳ thi CKA lẫn CKAD.

</p>
</details>

### Tạo một pod busybox (bằng lệnh kubectl) chạy lệnh "env". Chạy nó và xem output

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl run busybox --image=busybox --command --restart=Never -it --rm -- env # -it giúp xem được output, --rm sẽ xóa pod ngay khi nó kết thúc
# hoặc, chạy không có -it
kubectl run busybox --image=busybox --command --restart=Never -- env
# rồi xem log của nó
kubectl logs busybox
```

> 💡 **Bổ sung:** Mọi thứ sau `--` là lệnh/tham số cho container. Có `--command` thì chúng được ghi vào trường `command` (ghi đè `ENTRYPOINT` của image); không có `--command` thì chúng được ghi vào `args` (ghi đè `CMD`). Đây là điểm hay bị hỏi khi phân biệt `command`/`args` trong Pod spec.

</p>
</details>

### Tạo một pod busybox (bằng YAML) chạy lệnh "env". Chạy nó và xem output

<details><summary>Xem lời giải</summary>
<p>

```bash
# tạo một YAML template bằng lệnh này
kubectl run busybox --image=busybox --restart=Never --dry-run=client -o yaml --command -- env > envpod.yaml
# xem nội dung
cat envpod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - command:
    - env
    image: busybox
    name: busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
# apply rồi xem log
kubectl apply -f envpod.yaml
kubectl logs busybox
```

</p>
</details>

### Lấy YAML của một namespace mới tên 'myns' mà không tạo nó

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create namespace myns -o yaml --dry-run=client
```

</p>
</details>

### Tạo YAML cho một ResourceQuota mới tên 'myrq' với hard limit là 1 CPU, 1G memory và 2 pod mà không tạo nó

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2 --dry-run=client -o yaml
```

> 💡 **Bổ sung:** Lưu ý đơn vị: `1G` là 10^9 byte, còn `1Gi` là 2^30 byte. Khi namespace có ResourceQuota giới hạn `cpu`/`memory`, mọi Pod mới tạo trong đó phải khai báo requests/limits tương ứng (hoặc được LimitRange gán mặc định), nếu không sẽ bị từ chối.

</p>
</details>

### Lấy danh sách pod trên tất cả namespace

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl get po --all-namespaces
```
Hoặc:

```bash
kubectl get po -A
```
</p>
</details>

### Tạo một pod tên nginx dùng image nginx và mở port 80

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --port=80
```

> 💡 **Bổ sung:** `--port` chỉ khai báo `containerPort` trong Pod spec (mang tính thông tin), không tạo Service. Nếu muốn tạo luôn một Service ClusterIP đi kèm, thêm flag `--expose`: `kubectl run nginx --image=nginx --port=80 --expose`.

</p>
</details>

### Đổi image của pod sang nginx:1.24.0. Quan sát thấy container sẽ được restart ngay khi image được pull về

<details><summary>Xem lời giải</summary>
<p>

*Lưu ý*: Ban đầu cột `RESTARTS` nên là 0 (lý tưởng là vậy, nhưng thực tế có thể là bất kỳ số nào)

```bash
# kubectl set image POD/POD_NAME CONTAINER_NAME=IMAGE_NAME:TAG
kubectl set image pod/nginx nginx=nginx:1.24.0
kubectl describe po nginx # bạn sẽ thấy event 'Container will be killed and recreated'
kubectl get po nginx -w # theo dõi pod
```

*Lưu ý*: một lúc sau khi đổi image, bạn sẽ thấy giá trị cột `RESTARTS` tăng thêm 1, vì container đã được restart, như các event ở cuối output của lệnh `kubectl describe pod`:

```
Events:
  Type    Reason     Age                  From               Message
  ----    ------     ----                 ----               -------
[...]
  Normal  Killing    100s                 kubelet, node3     Container pod1 definition changed, will be restarted
  Normal  Pulling    100s                 kubelet, node3     Pulling image "nginx:1.24.0"
  Normal  Pulled     41s                  kubelet, node3     Successfully pulled image "nginx:1.24.0"
  Normal  Created    36s (x2 over 9m43s)  kubelet, node3     Created container pod1
  Normal  Started    36s (x2 over 9m43s)  kubelet, node3     Started container pod1
```

*Lưu ý*: bạn có thể kiểm tra image của pod bằng lệnh

```bash
kubectl get po nginx -o jsonpath='{.spec.containers[].image}{"\n"}'
```

> 💡 **Bổ sung:** Trường `image` là một trong số ít trường của Pod có thể sửa trực tiếp khi Pod đang chạy. Phần lớn các trường khác (env, ports, volumeMounts...) là bất biến, nên muốn đổi thì phải xóa Pod rồi tạo lại (`kubectl replace --force -f pod.yaml`). Với Deployment thì `kubectl set image deployment/<tên> ...` sẽ kích hoạt rolling update thay vì restart container tại chỗ.

</p>
</details>

### Lấy IP của pod nginx tạo ở bước trước, dùng một pod busybox tạm thời để wget đường dẫn '/' của nó

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl get po -o wide # lấy IP, sẽ có dạng '10.1.1.131'
# tạo một pod busybox tạm thời
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- 10.1.1.131:80
```

Bạn cũng có thể thử cách nâng cao hơn:

```bash
# Lấy IP của pod nginx
NGINX_IP=$(kubectl get pod nginx -o jsonpath='{.status.podIP}')
# tạo một pod busybox tạm thời
kubectl run busybox --image=busybox --env="NGINX_IP=$NGINX_IP" --rm -it --restart=Never -- sh -c 'wget -O- $NGINX_IP:80'
``` 

Hoặc gói gọn trong một dòng:

```bash
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- $(kubectl get pod nginx -o jsonpath='{.status.podIP}:{.spec.containers[0].ports[0].containerPort}')
```

> 💡 **Bổ sung:** Ở cách thứ hai, lệnh `sh -c` được đặt trong nháy đơn nên `$NGINX_IP` không bị shell trên máy bạn thay thế mà được shell bên trong container đọc từ biến môi trường truyền qua `--env`. Ở cách thứ ba thì ngược lại: `$(...)` được shell trên máy bạn thực thi trước khi gửi lệnh. Kỹ thuật "pod tạm thời + `wget`/`nc`" rất hay dùng trong CKA để kiểm tra Service và Network Policy.

</p>
</details>

### Lấy YAML của pod

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl get po nginx -o yaml
# hoặc
kubectl get po nginx -oyaml
# hoặc
kubectl get po nginx --output yaml
# hoặc
kubectl get po nginx --output=yaml
```

</p>
</details>

### Lấy thông tin về pod, bao gồm chi tiết các vấn đề có thể có (ví dụ pod chưa khởi động được)

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl describe po nginx
```

</p>
</details>

### Lấy log của pod

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl logs nginx
```

</p>
</details>

### Nếu pod bị crash và restart, lấy log của instance trước đó

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl logs nginx -p
# hoặc
kubectl logs nginx --previous
```

> 💡 **Bổ sung:** Với Pod ở trạng thái `CrashLoopBackOff`, `kubectl logs` thường chỉ cho thấy container hiện tại (vừa khởi động lại, có thể chưa ghi gì), còn `--previous` mới cho thấy lý do lần chạy trước bị chết. Kết hợp với `kubectl describe po` (xem `Last State`, `Exit Code`) là bộ đôi troubleshooting cơ bản trong CKA.

</p>
</details>

### Mở một shell đơn giản trên pod nginx

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl exec -it nginx -- /bin/sh
```

> 💡 **Bổ sung:** Nếu image không có shell (ví dụ image distroless), `kubectl exec` sẽ thất bại. Khi đó dùng ephemeral container (GA từ v1.25): `kubectl debug -it nginx --image=busybox --target=nginx` để gắn một container debug vào Pod đang chạy.

</p>
</details>

### Tạo một pod busybox in ra 'hello world' rồi thoát

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl run busybox --image=busybox -it --restart=Never -- echo 'hello world'
# hoặc
kubectl run busybox --image=busybox -it --restart=Never -- /bin/sh -c 'echo hello world'
```

</p>
</details>

### Làm tương tự, nhưng pod tự động bị xóa khi chạy xong

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl run busybox --image=busybox -it --rm --restart=Never -- /bin/sh -c 'echo hello world'
kubectl get po # không còn thấy đâu :)
```

</p>
</details>

### Tạo một pod nginx và đặt biến môi trường 'var1=val1'. Kiểm tra biến môi trường này có tồn tại trong pod hay không

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never --env=var1=val1
# sau đó
kubectl exec -it nginx -- env
# hoặc
kubectl exec -it nginx -- sh -c 'echo $var1'
# hoặc
kubectl describe po nginx | grep val1
# hoặc
kubectl run nginx --restart=Never --image=nginx --env=var1=val1 -it --rm -- env
# hoặc
kubectl run nginx --image nginx --restart=Never --env=var1=val1 -it --rm -- sh -c 'echo $var1'
```

</p>
</details>
