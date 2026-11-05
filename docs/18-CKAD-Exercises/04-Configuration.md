# Configuration (18%) - Cấu hình

[ConfigMaps](#configmaps)

[SecurityContext](#securitycontext)

[Resource Requests and Limits](#resource-requests-and-limits)

[Limit Ranges](#limit-ranges)

[Resource Quotas](#resource-quotas)

[Secrets](#secrets)

[Service Accounts](#serviceaccounts)

<br>#Mẹo: export sẵn thành biến<br>
<br>export ns="-n secret-ops"</br>
<br>export do="--dry-run=client -oyaml"</br>

> 💡 **Bổ sung:** Sau khi export, `k create ... $ns $do > file.yaml` sẽ chỉ sinh YAML ra file mà không tạo resource thật (`--dry-run=client -o yaml`). Đây là mẹo tiết kiệm thời gian rất phổ biến trong cả CKAD lẫn CKA. Trong môi trường thi, `alias k=kubectl` và bash completion thường đã được cấu hình sẵn. Nếu chưa có, bạn tự thêm vào `~/.bashrc`.

## ConfigMaps

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Pod to Use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

### Tạo ConfigMap tên config với các giá trị foo=lala,foo2=lolo

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create configmap config --from-literal=foo=lala --from-literal=foo2=lolo
```

</p>
</details>

### Hiển thị các giá trị của nó

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl get cm config -o yaml
# hoặc
kubectl describe cm config
```

</p>
</details>

### Tạo và hiển thị ConfigMap từ một file

Tạo file bằng lệnh

```bash
echo -e "foo3=lili\nfoo4=lele" > config.txt
```

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create cm configmap2 --from-file=config.txt
kubectl get cm configmap2 -o yaml
```

> 💡 **Bổ sung:** Với `--from-file`, **cả file** trở thành một key duy nhất. Tên key là tên file (`config.txt`), giá trị là toàn bộ nội dung file. Kubernetes không tách từng dòng `foo3=lili` thành key riêng.

</p>
</details>

### Tạo và hiển thị ConfigMap từ một file .env

Tạo file bằng lệnh

```bash
echo -e "var1=val1\n# this is a comment\n\nvar2=val2\n#anothercomment" > config.env
```

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create cm configmap3 --from-env-file=config.env
kubectl get cm configmap3 -o yaml
```

> 💡 **Bổ sung:** Khác với `--from-file`, `--from-env-file` đọc file theo định dạng `KEY=VALUE`: mỗi dòng thành một key riêng (`var1`, `var2`), còn dòng trống và dòng bắt đầu bằng `#` bị bỏ qua.

</p>
</details>

### Tạo và hiển thị ConfigMap từ một file, đặt tên key là 'special'

Tạo file bằng lệnh

```bash
echo -e "var3=val3\nvar4=val4" > config4.txt
```

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create cm configmap4 --from-file=special=config4.txt
kubectl describe cm configmap4
kubectl get cm configmap4 -o yaml
```

</p>
</details>

### Tạo ConfigMap tên 'options' với giá trị var5=val5. Tạo một Pod nginx mới, nạp giá trị của biến 'var5' vào biến môi trường tên 'option'

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create cm options --from-literal=var5=val5
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
    env:
    - name: option # tên biến môi trường
      valueFrom:
        configMapKeyRef:
          name: options # tên ConfigMap
          key: var5 # tên key trong ConfigMap
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env | grep option # sẽ hiển thị 'option=val5'
```

</p>
</details>

### Tạo ConfigMap 'anotherone' với các giá trị 'var6=val6', 'var7=val7'. Nạp toàn bộ ConfigMap này thành biến môi trường trong một Pod nginx mới

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create configmap anotherone --from-literal=var6=val6 --from-literal=var7=val7
kubectl run --restart=Never nginx --image=nginx -o yaml --dry-run=client > pod.yaml
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
    envFrom: # khác bài trước, bài trước dùng 'env'
    - configMapRef: # khác bài trước, bài trước dùng 'configMapKeyRef'
        name: anotherone # tên ConfigMap
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env 
```

> 💡 **Bổ sung:** `envFrom` nạp **mọi** key của ConfigMap thành biến môi trường cùng tên. Bạn có thể thêm `prefix: CFG_` để tránh trùng tên. Lưu ý: biến môi trường chỉ được đọc lúc container khởi động, nên khi ConfigMap thay đổi, Pod phải restart (hoặc được tạo lại) mới nhận giá trị mới.

</p>
</details>

### Tạo ConfigMap 'cmvolume' với các giá trị 'var8=val8', 'var9=val9'. Mount nó dưới dạng volume vào Pod nginx tại đường dẫn '/etc/lala'. Tạo Pod rồi chạy 'ls' trong thư mục '/etc/lala'.

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create configmap cmvolume --from-literal=var8=val8 --from-literal=var9=val9
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
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
  volumes: # thêm danh sách volumes
  - name: myvolume # chỉ là một cái tên, sẽ được tham chiếu lại trong container
    configMap:
      name: cmvolume # tên ConfigMap của bạn
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    volumeMounts: # liệt kê các volume mount ở đây
    - name: myvolume # tên đã khai báo ở pod.spec.volumes.name
      mountPath: /etc/lala # đường dẫn bên trong container
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- /bin/sh
cd /etc/lala
ls # sẽ hiển thị var8 var9
cat var8 # sẽ hiển thị val8
```

> 💡 **Bổ sung:** Khi mount ConfigMap dưới dạng volume, mỗi key thành một file. Khi ConfigMap được cập nhật, kubelet tự đồng bộ nội dung file (có độ trễ, thường dưới một phút). Ngoại lệ: volume mount bằng `subPath` **không** được cập nhật tự động. Nếu chỉ cần mount một số key, dùng `items` (gồm `key` và `path`) trong phần `configMap` của volume.

</p>
</details>

## SecurityContext

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

### Tạo YAML cho Pod nginx chạy với user ID 101. Không cần tạo Pod

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
  securityContext: # thêm dòng này
    runAsUser: 101 # UID của user
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

> 💡 **Bổ sung:** `securityContext` có hai cấp: cấp Pod (`spec.securityContext`) áp dụng cho mọi container, và cấp container (`spec.containers[].securityContext`). Nếu cả hai cùng đặt một trường (vd `runAsUser`), giá trị ở cấp container được ưu tiên. Một số trường chỉ có ở cấp Pod (`fsGroup`, `supplementalGroups`), một số chỉ có ở cấp container (`capabilities`, `privileged`, `allowPrivilegeEscalation`, `readOnlyRootFilesystem`). Kiểm tra nhanh bằng `kubectl explain pod.spec.securityContext` và `kubectl explain pod.spec.containers.securityContext`.

</p>
</details>


### Tạo YAML cho Pod nginx mà container duy nhất của nó được thêm các capability "NET_ADMIN", "SYS_TIME"

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
    securityContext: # thêm dòng này
      capabilities: # và dòng này
        add: ["NET_ADMIN", "SYS_TIME"] # cả dòng này nữa
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

> 💡 **Bổ sung:** `capabilities` chỉ khai báo được ở cấp **container**, không có ở cấp Pod. Tên capability viết không có tiền tố `CAP_` (viết `NET_ADMIN`, không viết `CAP_NET_ADMIN`). Ngoài `add`, còn có `drop` (vd `drop: ["ALL"]`), cấu hình hay gặp khi làm theo Pod Security Standards mức `restricted`.

</p>
</details>

## Resource requests and limits

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Assign CPU Resources to Containers and Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)

### Tạo Pod nginx với requests cpu=100m,memory=256Mi và limits cpu=200m,memory=512Mi

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
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
    name: nginx
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
      limits:    
        memory: "512Mi"
        cpu: "200m"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
``` 

> 💡 **Bổ sung:**
> - Các flag `--requests`/`--limits` của `kubectl run` đã bị loại bỏ trong các bản kubectl mới, nên cách chắc chắn là sinh YAML rồi sửa như trên. Với Deployment có sẵn, dùng `kubectl set resources deployment <tên> --requests=cpu=100m,memory=256Mi --limits=cpu=200m,memory=512Mi`.
> - `requests` là lượng tài nguyên scheduler dùng để chọn node. `limits` là mức trần lúc chạy: vượt CPU limit thì container bị throttle, vượt memory limit thì container bị kill (`OOMKilled`).
> - Pod có `requests` < `limits` như ví dụ này thuộc QoS class `Burstable`. Nếu mọi container có `requests` = `limits` cho cả CPU và memory thì Pod thuộc class `Guaranteed`, còn không đặt gì thì thuộc `BestEffort`. Xem bằng `kubectl get pod nginx -o jsonpath='{.status.qosClass}'`.
> - Liên hệ CKA: requests/limits ảnh hưởng trực tiếp tới scheduling (Pod `Pending` vì `Insufficient cpu/memory`), một lỗi hay gặp trong phần troubleshooting.

</p>
</details>

## Limit Ranges
kubernetes.io > Documentation > Concepts > Policies > Limit Ranges (https://kubernetes.io/docs/concepts/policy/limit-range/)

### Tạo namespace tên limitrange cùng một LimitRange giới hạn memory của Pod tối đa 500Mi và tối thiểu 100Mi

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create ns limitrange
```

vi 1.yaml
```YAML
apiVersion: v1
kind: LimitRange
metadata:
  name: ns-memory-limit
  namespace: limitrange
spec:
  limits:
  - max: # max và min xác định khoảng giới hạn
      memory: "500Mi"
    min:
      memory: "100Mi"
    type: Pod
```

```bash
kubectl apply -f 1.yaml
```

> 💡 **Bổ sung:** `type: Pod` giới hạn **tổng** tài nguyên của mọi container trong Pod. `type: Container` giới hạn từng container, và chỉ loại này hỗ trợ `default` (limit mặc định) và `defaultRequest` (request mặc định), được tự gán cho container không khai báo resources. LimitRange chỉ được kiểm tra khi tạo hoặc cập nhật Pod, không ảnh hưởng tới các Pod đang chạy.

</p>
</details>

### Describe namespace limitrange

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl describe limitrange ns-memory-limit -n limitrange
```

> 💡 **Bổ sung:** `kubectl describe ns limitrange` cũng hiển thị phần `Resource Limits` (và `Resource Quotas` nếu có) của namespace.

</p>
</details>

### Tạo Pod nginx request 250Mi memory trong namespace limitrange

<details><summary>Xem lời giải</summary>
<p>

vi 2.yaml
```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
  namespace: limitrange
spec:
  containers:
  - image: nginx
    name: nginx
    resources:
      requests:
        memory: "250Mi"
      limits:
        memory: "500Mi" # phải khai báo limit và limit phải <= mức max của LimitRange
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
``` 

```bash
kubectl apply -f 2.yaml
```

> 💡 **Bổ sung:** Vì sao bắt buộc khai báo limit? Với `max` ở `type: Pod`, nếu container không có memory limit thì limit của Pod được coi là không giới hạn, vượt `max` 500Mi, nên Pod bị từ chối với lỗi `Forbidden`. Nếu LimitRange có thêm mục `type: Container` với `default`, limit sẽ được tự gán và không cần khai báo thủ công.

</p>
</details>


## Resource Quotas
kubernetes.io > Documentation > Concepts > Policies > Resource Quotas (https://kubernetes.io/docs/concepts/policy/resource-quotas/)

### Tạo ResourceQuota trong namespace `one` với hard requests `cpu=1`, `memory=1Gi` và hard limits `cpu=2`, `memory=2Gi`.

<details><summary>Xem lời giải</summary>
<p>

Tạo namespace:
```bash
kubectl create ns one
```

Tạo ResourceQuota
```bash
vi rq-one.yaml
```

```YAML
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-rq
  namespace: one
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```

```bash
kubectl apply -f rq-one.yaml
```

hoặc
```bash
kubectl create quota my-rq --namespace=one --hard=requests.cpu=1,requests.memory=1Gi,limits.cpu=2,limits.memory=2Gi
```

> 💡 **Bổ sung:** LimitRange giới hạn **từng** Pod/container, còn ResourceQuota giới hạn **tổng** tài nguyên (và số lượng object như `pods`, `services`, `secrets`...) của cả namespace. Khi namespace có quota cho `requests.*`/`limits.*`, mọi Pod mới **bắt buộc** khai báo các giá trị tương ứng (hoặc được LimitRange gán mặc định), nếu không sẽ bị từ chối.

</p>
</details>

### Thử tạo Pod với resource requests `cpu=2`, `memory=3Gi` và limits `cpu=3`, `memory=4Gi` trong namespace `one`

<details><summary>Xem lời giải</summary>
<p>

```bash
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
  namespace: one
spec:
  containers:
  - image: nginx
    name: nginx
    resources:
      requests:
        memory: "3Gi"
        cpu: "2"
      limits:
        memory: "4Gi"
        cpu: "3"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl create -f pod.yaml
```

Thông báo lỗi dự kiến:
```bash
Error from server (Forbidden): error when creating "pod.yaml": pods "nginx" is forbidden: exceeded quota: my-rq, requested: limits.cpu=3,limits.memory=4Gi,requests.cpu=2,requests.memory=3Gi, used: limits.cpu=0,limits.memory=0,requests.cpu=0,requests.memory=0, limited: limits.cpu=2,limits.memory=2Gi,requests.cpu=1,requests.memory=1Gi
```

> 💡 **Bổ sung:** Với Pod tạo trực tiếp, lỗi quota hiện ngay khi chạy lệnh. Với Deployment, lệnh tạo vẫn thành công nhưng ReplicaSet không tạo được Pod. Khi đó cần xem `kubectl describe rs <tên>` hoặc `kubectl get events -n one` để thấy lỗi `exceeded quota`. Đây là tình huống troubleshooting hay gặp trong cả CKAD và CKA.

</p>
</details>

### Tạo Pod với resource requests `cpu=0.5`, `memory=1Gi` và limits `cpu=1`, `memory=2Gi` trong namespace `one`

<details><summary>Xem lời giải</summary>
<p>

```bash
vi pod2.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
  namespace: one
spec:
  containers:
  - image: nginx
    name: nginx
    resources:
      requests:
        memory: "1Gi"
        cpu: "0.5"
      limits:
        memory: "2Gi"
        cpu: "1"
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
kubectl create -f pod2.yaml
```

Xem mức sử dụng ResourceQuota trong namespace `one`
```bash
kubectl get resourcequota -n one
```

```
NAME    AGE   REQUEST                                          LIMIT
my-rq   10m   requests.cpu: 500m/1, requests.memory: 1Gi/1Gi   limits.cpu: 1/2, limits.memory: 2Gi/2Gi
```

> 💡 **Bổ sung:** `cpu: "0.5"` tương đương `500m` (milliCPU), đúng như cột `REQUEST` hiển thị. `kubectl describe quota my-rq -n one` cho bảng `Used`/`Hard` dễ đọc hơn.

</p>
</details>


## Secrets

kubernetes.io > Documentation > Concepts > Configuration > [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

kubernetes.io > Documentation > Tasks > Inject Data Into Applications > [Distribute Credentials Securely Using Secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)

### Tạo Secret tên mysecret với giá trị password=mypass

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create secret generic mysecret --from-literal=password=mypass
```

> 💡 **Bổ sung:** Dữ liệu trong Secret chỉ được **mã hóa base64**, không phải mật mã hóa. Ai có quyền `get secret` đều đọc được. Để bảo vệ Secret trong etcd cần bật encryption at rest (`--encryption-provider-config` trên kube-apiserver), một chủ đề thuộc kỳ thi CKA. Ngoài `generic`, `kubectl create secret` còn có `docker-registry` và `tls`.

</p>
</details>

### Tạo Secret tên mysecret2 lấy key/value từ một file

Tạo file tên username với giá trị admin:

```bash
echo -n admin > username
```

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create secret generic mysecret2 --from-file=username
```

> 💡 **Bổ sung:** `echo -n` để không ghi ký tự xuống dòng vào cuối file. Nếu thiếu `-n`, giá trị Secret sẽ là `admin\n`, một lỗi khó phát hiện khi ứng dụng dùng nó làm mật khẩu.

</p>
</details>

### Lấy giá trị của mysecret2

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl get secret mysecret2 -o yaml
echo -n YWRtaW4= | base64 -d # trên MAC là -D; lệnh này giải mã và hiển thị 'admin'
```

Cách khác, dùng `--jsonpath`:

```bash
kubectl get secret mysecret2 -o jsonpath='{.data.username}' | base64 -d  # trên MAC là -D
```

Cách khác, dùng `--template`:

```bash
kubectl get secret mysecret2 --template '{{.data.username}}' | base64 -d  # trên MAC là -D
```

Cách khác, dùng `jq`:

```bash
kubectl get secret mysecret2 -o json | jq -r .data.username | base64 -d  # trên MAC là -D
```

> 💡 **Bổ sung:** Các bản macOS gần đây cũng chấp nhận `base64 -d`. Môi trường thi là Linux nên luôn dùng `-d`. Đúng ra flag là `-o jsonpath=...` (như trong lệnh), không phải `--jsonpath`.

</p>
</details>

### Tạo Pod nginx mount Secret mysecret2 dưới dạng volume tại đường dẫn /etc/foo

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
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
  volumes: # khai báo các volume
  - name: foo # tên này dùng để tham chiếu bên trong container
    secret: # volume loại secret
      secretName: mysecret2 # tên Secret - Secret phải tồn tại sẵn khi tạo Pod
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
    volumeMounts: # các volume mount
    - name: foo # tên ở pod.spec.volumes
      mountPath: /etc/foo # đường dẫn mount
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- /bin/bash
ls /etc/foo  # hiển thị username
cat /etc/foo/username # hiển thị admin
```

> 💡 **Bổ sung:** Lưu ý tên trường: volume ConfigMap dùng `configMap.name`, còn volume Secret dùng `secret.secretName`. Nội dung file trong volume đã được giải mã base64 sẵn, và volume Secret được lưu trên tmpfs (RAM) của node, không ghi xuống đĩa.

</p>
</details>

### Xóa Pod vừa tạo và đưa key 'username' của Secret mysecret2 vào biến môi trường tên 'USERNAME' trong một Pod nginx mới

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl delete po nginx
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
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
    env: # các biến môi trường
    - name: USERNAME # tên theo yêu cầu đề bài
      valueFrom:
        secretKeyRef: # tham chiếu tới Secret
          name: mysecret2 # tên Secret
          key: username # key của dữ liệu trong Secret
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

```bash
kubectl create -f pod.yaml
kubectl exec -it nginx -- env | grep USERNAME | cut -d '=' -f 2 # sẽ hiển thị 'admin'
```

</p>
</details>

### Tạo Secret tên 'ext-service-secret' trong namespace 'secret-ops', với cặp key-value API_KEY=LmLHbYhsgWZwNifiqaRorH8T truyền dưới dạng literal.

<details><summary>Xem lời giải</summary>
<p>

```bash
export ns="-n secret-ops"
export do="--dry-run=client -oyaml"
k create secret generic ext-service-secret --from-literal=API_KEY=LmLHbYhsgWZwNifiqaRorH8T $ns $do > sc.yaml
k apply -f sc.yaml
```

> 💡 **Bổ sung:** Namespace `secret-ops` phải tồn tại trước. Nếu chưa có, tạo bằng `k create ns secret-ops`.

</p>
</details>

### Sử dụng Secret. Tạo Pod tên 'consumer' với image 'nginx' trong namespace 'secret-ops' và dùng Secret làm biến môi trường. Sau đó mở shell tương tác vào Pod và in ra mọi biến môi trường.
<details><summary>Xem lời giải</summary>
<p>

```bash
export ns="-n secret-ops"
export do="--dry-run=client -oyaml"
k run consumer --image=nginx $ns $do > nginx.yaml
vi nginx.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: consumer
  name: consumer
  namespace: secret-ops
spec:
  containers:
  - image: nginx
    name: consumer
    resources: {}
    env:
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: ext-service-secret
          key: API_KEY
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
k create $ns -f nginx.yaml
k exec -it $ns consumer -- /bin/sh
#env
```

> 💡 **Bổ sung:** Dòng `#env` nghĩa là gõ lệnh `env` tại dấu nhắc `#` của shell trong container. Cách nhanh hơn, không cần mở shell: `k exec $ns consumer -- env`. Muốn nạp mọi key của Secret thì dùng `envFrom` với `secretRef`, tương tự `configMapRef` ở phần ConfigMap.

</p>
</details>

### Tạo Secret tên 'my-secret' kiểu 'kubernetes.io/ssh-auth' trong namespace 'secret-ops'. Khai báo một key duy nhất tên 'ssh-privatekey', trỏ tới file 'id_rsa' trong thư mục hiện tại.
<details><summary>Xem lời giải</summary>
<p>

```bash
#Mẹo: export sẵn thành biến
export do="--dry-run=client -oyaml"
export ns="-n secret-ops"

#nếu file id_rsa chưa tồn tại.
ssh-keygen

k create secret generic my-secret $ns --type="kubernetes.io/ssh-auth" --from-file=ssh-privatekey=id_rsa $do > sc.yaml
k apply -f sc.yaml
```

> 💡 **Bổ sung:** Secret kiểu `kubernetes.io/ssh-auth` bắt buộc có key `ssh-privatekey`, nên phải dùng cú pháp `--from-file=<key>=<file>` để đặt đúng tên key. Mặc định `ssh-keygen` lưu khóa vào `~/.ssh/id_rsa` (hoặc `id_ed25519` ở bản OpenSSH mới), nên khi được hỏi đường dẫn hãy nhập `./id_rsa`, hoặc dùng `ssh-keygen -t rsa -f id_rsa -N ""`.

</p>
</details>

### Tạo Pod tên 'consumer' với image 'nginx' trong namespace 'secret-ops' và dùng Secret dưới dạng volume. Mount Secret vào đường dẫn /var/app ở chế độ chỉ đọc (read-only). Mở shell tương tác vào Pod và hiển thị nội dung file.
<details><summary>Xem lời giải</summary>
<p>

```bash
#Mẹo: export sẵn thành biến
export ns="-n secret-ops"
export do="--dry-run=client -oyaml"
k run consumer --image=nginx $ns $do > nginx.yaml
vi nginx.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: consumer
  name: consumer
  namespace: secret-ops
spec:
  containers:
    - image: nginx
      name: consumer
      resources: {}
      volumeMounts:
        - name: foo
          mountPath: "/var/app"
          readOnly: true
  volumes:
    - name: foo
      secret:
        secretName: my-secret
        optional: true
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
k exec -it $ns consumer -- /bin/sh
# cat /var/app/ssh-privatekey
# exit
```

> 💡 **Bổ sung:**
> - Lời giải bỏ qua bước tạo Pod: trước khi `exec`, cần chạy `k apply -f nginx.yaml`. Nếu Pod `consumer` từ bài trước vẫn còn, xóa nó trước bằng `k delete po consumer $ns`, vì không thể đổi volume của Pod đang chạy.
> - `optional: true` cho phép Pod vẫn khởi động khi Secret chưa tồn tại (volume sẽ rỗng). Đề bài không yêu cầu trường này. Bỏ đi thì Pod sẽ kẹt ở `ContainerCreating` nếu thiếu Secret, giúp phát hiện lỗi sớm hơn.

</p>
</details>

## ServiceAccounts

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

### Xem mọi ServiceAccount của cluster trong tất cả namespace

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl get sa --all-namespaces
```
Hoặc

```bash
kubectl get sa -A
```

</p>
</details>

### Tạo ServiceAccount mới tên 'myuser'

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create sa myuser
```

Cách khác:

```bash
# lấy template cho nhanh
kubectl get sa default -o yaml > sa.yaml
vim sa.yaml
```

```YAML
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myuser
```

```bash
kubectl create -f sa.yaml
```

> 💡 **Bổ sung:** Khi lấy template từ `default`, nhớ xóa các trường do hệ thống sinh (`uid`, `resourceVersion`, `creationTimestamp`). Gọn hơn là dùng `kubectl create sa myuser --dry-run=client -o yaml > sa.yaml`. Mỗi namespace luôn có sẵn ServiceAccount `default`, và Pod không khai báo ServiceAccount sẽ dùng nó.

</p>
</details>

### Tạo Pod nginx dùng ServiceAccount 'myuser'

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl run nginx --image=nginx --restart=Never -o yaml --dry-run=client > pod.yaml
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
  serviceAccountName: myuser # dùng pod.spec.serviceAccountName
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

hoặc

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  serviceAccount: myuser # dùng pod.spec.serviceAccount
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
kubectl get pod nginx -o jsonpath='{.spec.serviceAccountName}' # output: myuser

kubectl exec -it nginx -- cat /var/run/secrets/kubernetes.io/serviceaccount/token # kiểm tra token của ServiceAccount đã được mount vào Pod chưa. Từ Kubernetes 1.24+, token được project động vào Pod

```

> 💡 **Bổ sung:**
> - Trường `serviceAccount` đã **deprecated**, chỉ còn để tương thích ngược. Hãy luôn dùng `serviceAccountName`.
> - Không đổi được ServiceAccount của Pod đang chạy, phải xóa và tạo lại Pod. Với Deployment, có thể dùng `kubectl set serviceaccount deployment <tên> myuser`.
> - Từ v1.24, Kubernetes không còn tự tạo Secret chứa token cho mỗi ServiceAccount. Token trong Pod là projected token có thời hạn và được kubelet tự xoay vòng (rotate). Muốn tắt việc mount token, đặt `automountServiceAccountToken: false` ở Pod hoặc ServiceAccount.
> - Liên hệ CKA: ServiceAccount thường đi kèm RBAC. Kiểm tra quyền bằng `kubectl auth can-i list pods --as=system:serviceaccount:<namespace>:myuser`.

</p>
</details>

### Tạo API token cho ServiceAccount 'myuser'

<details><summary>Xem lời giải</summary>
<p>
  
```bash
kubectl create token myuser
```

> 💡 **Bổ sung:** `kubectl create token` (có từ v1.24) sinh token ngắn hạn qua TokenRequest API, mặc định có hiệu lực khoảng 1 giờ. Có thể đổi bằng `--duration=24h`, chọn namespace bằng `-n`. Token này không được lưu vào đâu cả. Nếu thật sự cần token dài hạn (cách cũ), tạo thủ công một Secret kiểu `kubernetes.io/service-account-token` có annotation `kubernetes.io/service-account.name: myuser`, và controller sẽ điền token vào Secret đó.

</p>
</details>
