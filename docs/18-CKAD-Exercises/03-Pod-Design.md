# Pod design - Thiết kế Pod (20%)

[Labels And Annotations](#labels-and-annotations)

[Deployments](#deployments)

[Jobs](#jobs)

[Cron Jobs](#cron-jobs)

## Labels and Annotations
kubernetes.io > Documentation > Concepts > Overview > Working with Kubernetes Objects > [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors)

> 💡 **Bổ sung:** Label dùng để **chọn** (select) đối tượng: Service, Deployment, ReplicaSet, Network Policy... đều tìm Pod qua label selector. Annotation chỉ để **gắn thông tin** (metadata không dùng để chọn), ví dụ người phụ trách, commit hash, cấu hình cho Ingress Controller. Vì vậy `kubectl get -l` chỉ lọc được theo label, không lọc được theo annotation.

### Tạo 3 Pod tên nginx1, nginx2, nginx3, tất cả đều có label app=v1

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl run nginx1 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx2 --image=nginx --restart=Never --labels=app=v1
kubectl run nginx3 --image=nginx --restart=Never --labels=app=v1
# hoặc
for i in `seq 1 3`; do kubectl run nginx$i --image=nginx -l app=v1 ; done
```

> 💡 **Bổ sung:** Từ v1.18, `kubectl run` chỉ tạo Pod (flag `--generator` đã bị loại bỏ), nên `--restart=Never` ở đây không bắt buộc. `-l` là dạng viết tắt của `--labels`. Nếu cần nhiều label, ngăn cách bằng dấu phẩy: `--labels=app=v1,tier=web`.

</p>
</details>

### Hiển thị tất cả label của các Pod

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl get po --show-labels
```

</p>
</details>

### Đổi label của Pod 'nginx2' thành app=v2

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl label po nginx2 app=v2 --overwrite
# hoặc sửa YAML của Pod
kubectl edit po nginx2
```

> 💡 **Bổ sung:** Không có `--overwrite`, kubectl sẽ báo lỗi `'app' already has a value (v1)` vì label `app` đã tồn tại. Cờ này chỉ cần khi đổi giá trị của label đã có, còn thêm label mới thì không cần.

</p>
</details>

### Lấy label 'app' của các Pod (hiển thị một cột APP)

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl get po -L app
# hoặc
kubectl get po --label-columns=app
```

> 💡 **Bổ sung:** Lưu ý phân biệt: `-L` (chữ hoa) **thêm cột** hiển thị giá trị label, còn `-l` (chữ thường) **lọc** theo label selector.

</p>
</details>

### Chỉ lấy các Pod có 'app=v2'

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl get po -l app=v2
# hoặc
kubectl get po -l 'app in (v2)'
# hoặc
kubectl get po --selector=app=v2
```

</p>
</details>

### Lấy các Pod có 'app=v2' nhưng không có 'tier=frontend'

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl get po -l app=v2,tier!=frontend
# hoặc
kubectl get po -l 'app in (v2), tier notin (frontend)'
# hoặc
kubectl get po --selector=app=v2,tier!=frontend
```

> 💡 **Bổ sung:** Dấu phẩy trong selector nghĩa là **AND**. Có hai kiểu selector: *equality-based* (`=`, `==`, `!=`) và *set-based* (`in`, `notin`, `key` để kiểm tra label có tồn tại, `!key` để kiểm tra không tồn tại). Nên đặt selector trong dấu nháy đơn để shell không hiểu nhầm `!` hay dấu ngoặc.

</p>
</details>

### Thêm label mới tier=web cho tất cả Pod có label 'app=v2' hoặc 'app=v1'

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl label po -l "app in(v1,v2)" tier=web
```
</p>
</details>


### Thêm annotation 'owner: marketing' cho tất cả Pod có label 'app=v2'

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl annotate po -l "app=v2" owner=marketing
```
</p>
</details>

### Xóa label 'app' khỏi các Pod đã tạo ở trên

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl label po nginx1 nginx2 nginx3 app-
# hoặc
kubectl label po nginx{1..3} app-
# hoặc
kubectl label po -l app app-
```

> 💡 **Bổ sung:** Dấu `-` ở cuối tên key (`app-`) nghĩa là xóa label đó. Cú pháp này cũng áp dụng cho annotation (`kubectl annotate ... owner-`) và taint (`kubectl taint node node1 tier=frontend:NoSchedule-`). `nginx{1..3}` là brace expansion của bash, được shell mở rộng thành `nginx1 nginx2 nginx3` trước khi chạy kubectl.

</p>
</details>

### Gắn annotation "description='my description'" cho các Pod nginx1, nginx2, nginx3

<details><summary>Xem lời giải</summary>
<p>


```bash
kubectl annotate po nginx1 nginx2 nginx3 description='my description'

# hoặc

kubectl annotate po nginx{1..3} description='my description'
```

</p>
</details>

### Kiểm tra annotation của Pod nginx1

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl annotate pod nginx1 --list

# hoặc

kubectl describe po nginx1 | grep -i 'annotations'

# hoặc

kubectl get po nginx1 -o custom-columns=Name:metadata.name,ANNOTATIONS:metadata.annotations.description
```

Thay vì dùng `| grep`, bạn có thể dùng jsonPath, ví dụ `kubectl get po nginx1 -o jsonpath='{.metadata.annotations}{"\n"}'`

</p>
</details>

### Xóa annotation của ba Pod này

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl annotate po nginx{1..3} description- owner-
```

</p>
</details>

### Xóa các Pod này để cluster trở lại trạng thái sạch

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl delete po nginx{1..3}
```

> 💡 **Bổ sung:** Khi luyện tập, thêm `--grace-period=0 --force` (hoặc `--now`) để xóa nhanh, khỏi phải chờ khoảng 30 giây graceful termination. Trong phòng thi, mẹo này giúp tiết kiệm thời gian, nhưng không nên dùng với Pod của StatefulSet trên môi trường thật.

</p>
</details>

## Pod Placement

> 💡 **Bổ sung:** Nhóm bài này (nodeSelector, node affinity, nodeName, taint/toleration) cũng thuộc phần *Scheduling* của kỳ thi CKA. Hãy nhớ: **nodeSelector/affinity** kéo Pod *về phía* node phù hợp, còn **taint** đẩy Pod *ra khỏi* node. Toleration chỉ *cho phép* Pod chạy trên node bị taint chứ không *buộc* Pod phải lên node đó. Muốn ép Pod lên đúng một node bị taint thì phải kết hợp cả hai, như bài cuối của mục này.

### Tạo một Pod sẽ được triển khai lên node có label 'accelerator=nvidia-tesla-p100'

<details><summary>Xem lời giải</summary>
<p>

Gắn label cho node:

```bash
kubectl label nodes <your-node-name> accelerator=nvidia-tesla-p100
kubectl get nodes --show-labels
```

Dùng thuộc tính 'nodeSelector' trong YAML của Pod:

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
    - name: cuda-test
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
  nodeSelector: # thêm phần này
    accelerator: nvidia-tesla-p100 # label dùng để chọn node
```

Bạn có thể dễ dàng biết nên đặt trường này ở đâu trong YAML bằng lệnh:

```bash
kubectl explain po.spec
```

HOẶC:
Dùng node affinity (https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/#schedule-a-pod-using-required-node-affinity)

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: affinity-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: accelerator
            operator: In
            values:
            - nvidia-tesla-p100
  containers:
    ...
```

> 💡 **Bổ sung:**
> - Registry `k8s.gcr.io` đã bị đóng băng từ tháng 4/2023; image chính thức của Kubernetes giờ nằm ở `registry.k8s.io`. Khi luyện tập, chỉ cần đổi image thành `nginx`, vì nội dung chính của bài là `nodeSelector`.
> - Nếu không có node nào khớp label, Pod sẽ nằm ở trạng thái `Pending` và `kubectl describe po` sẽ hiện event `FailedScheduling ... didn't match Pod's node affinity/selector`.
> - `kubectl explain po.spec.affinity.nodeAffinity --recursive` hiển thị toàn bộ cấu trúc trường, rất hữu ích khi không nhớ cú pháp lúc thi. `requiredDuringScheduling...` là điều kiện bắt buộc, còn `preferredDuringScheduling...` chỉ là ưu tiên (có `weight`). Phần `IgnoredDuringExecution` nghĩa là nếu sau này label của node thay đổi, Pod đang chạy cũng không bị đuổi đi.

</p>
</details>

### Tạo một Pod sẽ được đặt lên node `node01` bằng `nodeName`

<details><summary>Xem lời giải</summary>
<p>

`nodeName` buộc Pod gắn vào một node cụ thể (bỏ qua scheduler). Chi tiết xem tài liệu chính thức: [https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodename](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodename)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nodename-pod
spec:
  nodeName: node01
  containers:
  - name: nodename-con
    image: nginx  
```

Kiểm tra Pod đã chạy trên node nào:

```bash
kubectl get pod nodename-pod -o wide
```

> 💡 **Bổ sung:** Vì bỏ qua scheduler nên `nodeName` cũng bỏ qua taint có effect `NoSchedule`, nhưng kubelet vẫn kiểm tra tài nguyên của node và Pod vẫn có thể bị đuổi bởi taint `NoExecute`. Nếu tên node sai hoặc không tồn tại, Pod sẽ không bao giờ chạy. Trong CKA, `nodeName` hay được dùng để "lập lịch thủ công" khi scheduler bị hỏng (một dạng bài troubleshooting quen thuộc). Lưu ý `nodeName` chỉ đặt được lúc tạo Pod, không sửa được trên Pod đang tồn tại.

</p>
</details>

### Taint một node với key `tier`, value `frontend` và effect `NoSchedule`. Sau đó tạo một Pod chịu được (tolerate) taint này.

<details><summary>Xem lời giải</summary>
<p>

Taint node:

```bash
kubectl taint node node1 tier=frontend:NoSchedule # key=value:Effect
kubectl describe node node1 # xem các taint trên node
```

Và để tolerate taint đó:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "tier"
    operator: "Equal"
    value: "frontend"
    effect: "NoSchedule"
```

> 💡 **Bổ sung:** Có ba effect: `NoSchedule` (không lập lịch Pod mới), `PreferNoSchedule` (cố gắng tránh) và `NoExecute` (đuổi cả Pod đang chạy nếu không tolerate; có thể dùng `tolerationSeconds` để trì hoãn). Xem nhanh taint: `kubectl describe node node1 | grep -i taint`. Gỡ taint: `kubectl taint node node1 tier=frontend:NoSchedule-`.

</p>
</details>

### Tạo một Pod sẽ được đặt lên node `controlplane`. Dùng nodeSelector và tolerations.

<details><summary>Xem lời giải</summary>
<p>

```bash
vi pod.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    kubernetes.io/hostname: controlplane
  tolerations:
  - key: "node-role.kubernetes.io/control-plane"
    operator: "Exists"
    effect: "NoSchedule"
```

```bash
kubectl create -f pod.yaml
```

> 💡 **Bổ sung:** Từ v1.25, kubeadm chỉ còn dùng taint `node-role.kubernetes.io/control-plane:NoSchedule`; taint cũ `node-role.kubernetes.io/master` đã bị bỏ. Trên cluster cũ hơn, hãy kiểm tra taint thực tế bằng `kubectl describe node controlplane | grep -i taint` trước khi viết toleration. `kubernetes.io/hostname` là label có sẵn trên mọi node, xem bằng `kubectl get node controlplane --show-labels`.

</p>
</details>

## Deployments

kubernetes.io > Documentation > Concepts > Workloads > Workload Resources > [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment)

### Tạo một Deployment tên nginx với image nginx:1.18.0, 2 replica, khai báo port 80 là port mà container này expose (không tạo Service cho Deployment này)

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create deployment nginx  --image=nginx:1.18.0  --dry-run=client -o yaml > deploy.yaml
vi deploy.yaml
# đổi trường replicas từ 1 thành 2
# thêm đoạn sau vào phần spec của container rồi lưu file deploy.yaml
# ports:
#   - containerPort: 80
kubectl apply -f deploy.yaml
```

hoặc làm như sau:

```bash
kubectl create deployment nginx  --image=nginx:1.18.0  --dry-run=client -o yaml | sed 's/replicas: 1/replicas: 2/g'  | sed 's/image: nginx:1.18.0/image: nginx:1.18.0\n        ports:\n        - containerPort: 80/g' | kubectl apply -f -
```

hoặc,
```bash
kubectl create deploy nginx --image=nginx:1.18.0 --replicas=2 --port=80
```

> 💡 **Bổ sung:** Cách cuối là nhanh nhất khi thi: `kubectl create deployment` hỗ trợ `--replicas` và `--port` (port này chỉ đặt `containerPort`, không tạo Service). Lưu ý cách dùng `sed` với `\n` chỉ hoạt động trên GNU sed (Linux, như môi trường thi); `sed` mặc định trên macOS không hiểu `\n` trong chuỗi thay thế.

</p>
</details>

### Xem YAML của Deployment này

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl get deploy nginx -o yaml
```

</p>
</details>

### Xem YAML của ReplicaSet do Deployment này tạo ra

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl describe deploy nginx # tên ReplicaSet nằm trong phần Events và ở thuộc tính 'NewReplicaSet'
# HOẶC tìm trực tiếp rs bằng:
kubectl get rs -l app=nginx
# hoặc chỉ cần chạy kubectl get rs
kubectl get rs nginx-7bf7478b77 -o yaml
```

> 💡 **Bổ sung:** Tên ReplicaSet có dạng `<tên-deployment>-<pod-template-hash>`; phần hash (ví dụ `7bf7478b77`) sẽ khác trên cluster của bạn. ReplicaSet và Pod đều mang label `pod-template-hash`, nên mỗi lần đổi template sẽ sinh ra một ReplicaSet mới.

</p>
</details>

### Lấy YAML của một trong các Pod

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl get po # liệt kê tất cả Pod
# HOẶC tìm trực tiếp Pod bằng:
kubectl get po -l run=nginx # nếu bạn tạo deployment bằng lệnh 'run'
kubectl get po -l app=nginx # nếu bạn tạo deployment bằng lệnh 'create'
kubectl get po nginx-7bf7478b77-gjzp8 -o yaml
```

> 💡 **Bổ sung:** Từ v1.18, `kubectl run` không còn tạo Deployment nữa, nên với cluster hiện tại chỉ còn dùng `-l app=nginx` (label do `kubectl create deployment` tự gắn).

</p>
</details>

### Kiểm tra tiến trình rollout của Deployment

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl rollout status deploy nginx
```

</p>
</details>

### Cập nhật image nginx thành nginx:1.19.8

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.19.8
# hoặc...
kubectl edit deploy nginx # sửa .spec.template.spec.containers[0].image
```

Cú pháp của lệnh 'kubectl set image' là `kubectl set image (-f FILENAME | TYPE NAME) CONTAINER_NAME_1=CONTAINER_IMAGE_1 ... CONTAINER_NAME_N=CONTAINER_IMAGE_N [options]`

> 💡 **Bổ sung:** Vế trái của `nginx=nginx:1.19.8` là **tên container** chứ không phải tên Deployment (ở đây trùng nhau vì `kubectl create deployment` đặt tên container theo tên image). Nếu đề yêu cầu ghi lại lý do thay đổi trong `rollout history`, đừng dùng `--record` (đã deprecated). Thay vào đó hãy gắn annotation: `kubectl annotate deploy nginx kubernetes.io/change-cause="update to 1.19.8"`; giá trị này sẽ hiện ở cột `CHANGE-CAUSE`.

</p>
</details>

### Kiểm tra lịch sử rollout và xác nhận các replica đều ổn

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl rollout history deploy nginx
kubectl get deploy nginx
kubectl get rs # kiểm tra rằng một ReplicaSet mới đã được tạo
kubectl get po
```

</p>
</details>

### Hoàn tác (undo) lần rollout gần nhất và xác nhận các Pod mới dùng image cũ (nginx:1.18.0)

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl rollout undo deploy nginx
# chờ một chút
kubectl get po # chọn một Pod đang 'Running'
kubectl describe po nginx-5ff4457d65-nslcl | grep -i image # phải là nginx:1.18.0
```

> 💡 **Bổ sung:** `rollout undo` không "quay lùi" số revision, mà lấy template của revision cũ và tạo thành **revision mới**; revision cũ đó biến mất khỏi danh sách. Trong chuỗi bài này: rev 1 = `1.18.0`, rev 2 = `1.19.8`, undo → rev 3 (`1.18.0`, rev 1 biến mất), image sai → rev 4 (`1.91`), undo về rev 2 → rev 5 (`1.19.8`, rev 2 biến mất), pause rồi đổi sang `1.19.9` và resume → rev 6. Đó là lý do các bài sau xem `--revision=4` (image sai) và `--revision=6`.

</p>
</details>

### Cố ý cập nhật Deployment với image sai nginx:1.91

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.91
# hoặc
kubectl edit deploy nginx
# đổi image thành nginx:1.91
# mẹo vim: gõ (không có dấu nháy) '/image' rồi nhấn Enter để nhảy nhanh tới dòng đó
```

</p>
</details>

### Xác nhận rằng rollout đang gặp vấn đề

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl rollout status deploy nginx
# hoặc
kubectl get po # bạn sẽ thấy 'ErrImagePull' hoặc 'ImagePullBackOff'
```

> 💡 **Bổ sung:** Nhờ chiến lược `RollingUpdate` (mặc định `maxUnavailable: 25%`, `maxSurge: 25%`), các Pod cũ vẫn tiếp tục chạy trong khi Pod mới bị lỗi pull image, nên ứng dụng không bị gián đoạn hoàn toàn. `kubectl rollout status` sẽ treo chờ; nhấn Ctrl+C để thoát hoặc thêm `--timeout=30s`. Sau `progressDeadlineSeconds` (mặc định 600 giây), Deployment sẽ có condition `Progressing=False` với reason `ProgressDeadlineExceeded`.

</p>
</details>


### Đưa Deployment về revision thứ hai (số 2) và xác nhận image là nginx:1.19.8

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl rollout undo deploy nginx --to-revision=2
kubectl describe deploy nginx | grep Image:
kubectl rollout status deploy nginx # Mọi thứ đều phải OK
```

</p>
</details>

### Xem chi tiết revision thứ tư (số 4)

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl rollout history deploy nginx --revision=4 # Bạn cũng sẽ thấy image sai hiển thị ở đây
```

</p>
</details>

### Scale Deployment lên 5 replica

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl scale deploy nginx --replicas=5
kubectl get po
kubectl describe deploy nginx
```

> 💡 **Bổ sung:** Scale không tạo revision mới, vì `replicas` không nằm trong Pod template. Chỉ khi thay đổi `.spec.template` thì mới sinh rollout mới.

</p>
</details>

### Autoscale Deployment với số Pod từ 5 đến 10, CPU utilization mục tiêu 80%

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl autoscale deploy nginx --min=5 --max=10 --cpu-percent=80
# xem horizontalpodautoscalers.autoscaling của nginx
kubectl get hpa nginx
```

> 💡 **Bổ sung:** Để HPA thực sự hoạt động, cluster cần có **metrics-server**, và container phải khai báo `resources.requests.cpu` (phần trăm CPU được tính dựa trên request). Nếu thiếu, cột `TARGETS` sẽ hiện `<unknown>/80%`. API ổn định hiện nay là `autoscaling/v2` (`autoscaling/v2beta2` đã bị gỡ ở v1.26). Autoscaling workload cũng nằm trong curriculum CKA từ năm 2025.

</p>
</details>

### Tạm dừng (pause) rollout của Deployment

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl rollout pause deploy nginx
```

</p>
</details>

### Cập nhật image thành nginx:1.19.9 và xác nhận rằng không có gì xảy ra, vì rollout đang bị tạm dừng

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl set image deploy nginx nginx=nginx:1.19.9
# hoặc
kubectl edit deploy nginx
# đổi image thành nginx:1.19.9
kubectl rollout history deploy nginx # không có revision mới
```

> 💡 **Bổ sung:** Khi đang pause, bạn có thể dồn nhiều thay đổi (image, resources, env...) rồi resume một lần để chỉ tạo **một** rollout duy nhất. Lưu ý: không thể `rollout undo` một Deployment đang bị pause.

</p>
</details>

### Tiếp tục (resume) rollout và xác nhận image nginx:1.19.9 đã được áp dụng

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl rollout resume deploy nginx
kubectl rollout history deploy nginx
kubectl rollout history deploy nginx --revision=6 # điền số revision mới nhất của bạn
```

</p>
</details>

### Xóa Deployment và horizontal pod autoscaler đã tạo

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl delete deploy nginx
kubectl delete hpa nginx

# hoặc
kubectl delete deploy/nginx hpa/nginx
```
</p>
</details>

### Triển khai canary deployment bằng cách chạy hai phiên bản nginx được đánh dấu version=v1 và version=v2, sao cho tải được phân bổ theo tỉ lệ 75%-25%

<details><summary>Xem lời giải</summary>
<p>

Triển khai 3 replica của v1:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v1
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: v1
  template:
    metadata:
      labels:
        app: my-app
        version: v1
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: workdir
          mountPath: /usr/share/nginx/html
      initContainers:
      - name: install
        image: busybox:1.28
        command:
        - /bin/sh
        - -c
        - "echo version-1 > /work-dir/index.html"
        volumeMounts:
        - name: workdir
          mountPath: "/work-dir"
      volumes:
      - name: workdir
        emptyDir: {}
```

Tạo Service:
```
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
  labels:
    app: my-app
spec:
  type: ClusterIP
  ports:
  - name: http
    port: 80
    targetPort: 80
  selector:
    app: my-app
```

Kiểm tra Deployment đã chạy thành công từ bên trong một Pod:
```
# chạy wget tới Service my-app-svc
kubectl run -it --rm --restart=Never busybox --image=gcr.io/google-containers/busybox --command -- wget -qO- my-app-svc

version-1
```

Triển khai 1 replica của v2:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v2
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
      version: v2
  template:
    metadata:
      labels:
        app: my-app
        version: v2
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: workdir
          mountPath: /usr/share/nginx/html
      initContainers:
      - name: install
        image: busybox:1.28
        command:
        - /bin/sh
        - -c
        - "echo version-2 > /work-dir/index.html"
        volumeMounts:
        - name: workdir
          mountPath: "/work-dir"
      volumes:
      - name: workdir
        emptyDir: {}
```

Quan sát thấy khi gọi tới IP mà Service expose, các request được cân bằng tải giữa hai phiên bản:
```
# chạy một Pod busybox liên tục gọi wget tới Service my-app-svc và in ra version của Pod mà request tới được.
kubectl run -it --rm --restart=Never busybox --image=gcr.io/google-containers/busybox -- /bin/sh -c 'while sleep 1; do wget -qO- my-app-svc; done'

version-1
version-1
version-1
version-2
version-2
version-1
```

Nếu v2 ổn định, scale nó lên 4 replica và tắt v1:
```
kubectl scale --replicas=4 deploy my-app-v2
kubectl delete deploy my-app-v1
while sleep 0.1; do curl $(kubectl get svc my-app-svc -o jsonpath="{.spec.clusterIP}"); done
version-2
version-2
version-2
version-2
version-2
version-2
```

> 💡 **Bổ sung:**
> - Mấu chốt của bài: Service chỉ chọn theo `app: my-app` (không có `version`), nên nó gom Pod của **cả hai** Deployment. Tỉ lệ 75/25 đến từ số replica 3:1. Đây chỉ là tỉ lệ *xấp xỉ*, vì kube-proxy chọn endpoint ngẫu nhiên (chế độ iptables). Muốn chia traffic chính xác theo phần trăm thì cần Gateway API (`HTTPRoute` với `weight`), service mesh hoặc Ingress Controller có hỗ trợ canary.
> - Image `gcr.io/google-containers/busybox` đã cũ; khi luyện tập chỉ cần dùng `--image=busybox`.
> - Lệnh `curl` tới ClusterIP ở bước cuối chỉ chạy được khi đứng **bên trong** cluster (trên một node hoặc trong một Pod), vì ClusterIP không truy cập được từ máy bên ngoài.

</p>
</details>

## Jobs

> 💡 **Bổ sung:** Tài liệu tham khảo: kubernetes.io > Documentation > Concepts > Workloads > Workload Resources > [Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/). Job chỉ chấp nhận `restartPolicy` là `Never` hoặc `OnFailure` (không được dùng `Always`).

### Tạo một Job tên pi với image perl:5.34, chạy lệnh có tham số "perl -Mbignum=bpi -wle 'print bpi(2000)'"

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create job pi  --image=perl:5.34 -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```

</p>
</details>

### Chờ Job chạy xong rồi lấy output

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl get jobs -w # chờ tới khi 'SUCCESSFUL' bằng 1 (sẽ mất một lúc, image perl khá lớn)
kubectl get po # lấy tên Pod
kubectl logs pi-**** # xem các chữ số của pi
kubectl delete job pi
```
HOẶC

```bash
kubectl get jobs -w # chờ tới khi 'SUCCESSFUL' bằng 1 (sẽ mất một lúc, image perl khá lớn)
kubectl logs job/pi
kubectl delete job pi
```
HOẶC

```bash
kubectl wait --for=condition=complete --timeout=300s job pi
kubectl logs job/pi
kubectl delete job pi
```

> 💡 **Bổ sung:** Ở các phiên bản kubectl mới, cột trạng thái của `kubectl get jobs` là `COMPLETIONS` (ví dụ `1/1`) và có thêm cột `STATUS` (`Running`, `Complete`, `Failed`) thay vì `SUCCESSFUL`. `kubectl wait` rất tiện khi viết script hoặc khi thi, vì lệnh tự trả về ngay khi Job hoàn tất.

</p>
</details>

### Tạo một Job với image busybox chạy lệnh 'echo hello;sleep 30;echo world'

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create job busybox --image=busybox -- /bin/sh -c 'echo hello;sleep 30;echo world'
```

</p>
</details>

### Theo dõi log của Pod (bạn sẽ phải chờ 30 giây)

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl get po # tìm Pod của Job
kubectl logs busybox-ptx58 -f # theo dõi log
```

</p>
</details>

### Xem trạng thái của Job, describe nó và xem log

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl get jobs
kubectl describe jobs busybox
kubectl logs job/busybox
```

</p>
</details>

### Xóa Job

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl delete job busybox
```

</p>
</details>

### Tạo lại Job tương tự, cho nó chạy 5 lần, lần lượt từng lần một. Kiểm tra trạng thái rồi xóa nó

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create job busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'echo hello;sleep 30;echo world' > job.yaml
vi job.yaml
```

Thêm job.spec.completions=5 và job.spec.completionMode=Indexed

```YAML
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  completions: 5 # thêm dòng này
  completionMode: Indexed # thêm dòng này
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - echo hello;sleep 30;echo world
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```

```bash
kubectl create -f job.yaml
```

Xác nhận Job đã hoàn tất:

```bash
kubectl get job busybox -w # sẽ mất khoảng hai phút rưỡi
kubectl delete jobs busybox
```

> 💡 **Bổ sung:** Để chạy tuần tự, chỉ cần `completions: 5`, vì `parallelism` mặc định là 1. `completionMode: Indexed` là tùy chọn: mỗi Pod sẽ nhận một chỉ số từ 0 đến 4 (qua biến môi trường `JOB_COMPLETION_INDEX` và annotation `batch.kubernetes.io/job-completion-index`), hữu ích khi mỗi Pod cần xử lý một phần dữ liệu riêng. Mặc định là `NonIndexed`. YAML do `kubectl create job` sinh ra có thể khác đôi chút tùy phiên bản (ví dụ `restartPolicy: Never`); cả `Never` và `OnFailure` đều hợp lệ cho Job.

</p>
</details>

### Tạo lại Job tương tự, nhưng cho nó chạy song song 5 lần

<details><summary>Xem lời giải</summary>
<p>

```bash
vi job.yaml
```

Thêm job.spec.parallelism=5

```YAML
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  parallelism: 5 # thêm dòng này
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - echo hello;sleep 30;echo world
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```

```bash
kubectl create -f job.yaml
kubectl get jobs
```

Các Job song song sẽ mất một khoảng thời gian để chạy xong (>= 30 giây)

```bash
kubectl delete job busybox
```

> 💡 **Bổ sung:** Nếu chỉ đặt `parallelism: 5` mà không có `completions`, Job hoạt động theo kiểu *work queue*: 5 Pod chạy cùng lúc và Job được coi là hoàn tất khi có ít nhất một Pod thành công và tất cả Pod đã kết thúc. Nếu đề yêu cầu "chạy 5 lần, mỗi lúc chạy song song tối đa N Pod", hãy đặt cả `completions: 5` lẫn `parallelism: N`.

</p>
</details>

### Tạo một Job nhưng đảm bảo nó sẽ bị Kubernetes tự động dừng nếu chạy quá 30 giây

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create job busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'while true; do echo hello; sleep 10;done' > job.yaml
vi job.yaml
```

Thêm job.spec.activeDeadlineSeconds=30

```bash
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  activeDeadlineSeconds: 30 # thêm dòng này
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: busybox
    spec:
      containers:
      - args:
        - /bin/sh
        - -c
        - while true; do echo hello; sleep 10;done
        image: busybox
        name: busybox
        resources: {}
      restartPolicy: OnFailure
status: {}
```

> 💡 **Bổ sung:** Khi vượt `activeDeadlineSeconds`, mọi Pod đang chạy của Job bị dừng và Job chuyển sang trạng thái `Failed` với reason `DeadlineExceeded` (xem bằng `kubectl describe job busybox`). `activeDeadlineSeconds` được ưu tiên hơn `backoffLimit` (số lần thử lại, mặc định 6). Hai trường liên quan khác cũng hay gặp: `backoffLimit` và `ttlSecondsAfterFinished` (tự xóa Job sau khi chạy xong N giây).

</p>
</details>

## Cron jobs

kubernetes.io > Documentation > Tasks > Run Jobs > [Running Automated Tasks with a CronJob](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/)

> 💡 **Bổ sung:** CronJob dùng `batch/v1` từ v1.21; `batch/v1beta1` đã bị gỡ ở v1.25, nên các ví dụ cũ dùng `v1beta1` sẽ không apply được nữa. Cú pháp `schedule` gồm 5 trường: `phút giờ ngày-trong-tháng tháng ngày-trong-tuần`, ví dụ `*/1 * * * *` (hoặc `* * * * *`) là mỗi phút. Từ v1.27, có thể đặt múi giờ bằng `spec.timeZone` (ví dụ `"Asia/Ho_Chi_Minh"`).

### Tạo một CronJob với image busybox chạy theo lịch "*/1 * * * *" và ghi 'date; echo Hello from the Kubernetes cluster' ra standard output

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create cronjob busybox --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'
```

</p>
</details>

### Xem log của nó rồi xóa nó

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl get po # copy ID của Pod có container vừa được tạo
kubectl logs <busybox-***> # bạn sẽ thấy ngày giờ và thông điệp
kubectl delete cj busybox # cj là viết tắt của cronjob
```

</p>
</details>

### Tạo lại CronJob đó và theo dõi trạng thái. Khi nó đã chạy, kiểm tra Job nào đã được CronJob tạo ra. Xem log rồi xóa CronJob

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl get cj
kubectl get jobs --watch
kubectl get po --show-labels # để ý rằng các Pod có label chỉ ra Job 'cha' của chúng
kubectl logs busybox-1529745840-m867r
# Lưu ý rằng Kubernetes sẽ chạy một Job/Pod mới cho mỗi lần CronJob được kích hoạt
kubectl delete cj busybox
```

> 💡 **Bổ sung:** Tên Job do CronJob tạo có dạng `<tên-cronjob>-<số>`, trong đó số là thời điểm lên lịch tính bằng phút kể từ Unix epoch (các phiên bản cũ dùng giây, như ví dụ ở trên). Pod mang label `job-name=<tên-job>` (từ v1.27 có thêm `batch.kubernetes.io/job-name`), nên có thể lọc bằng `kubectl get po -l job-name=<tên-job>`. Xóa CronJob sẽ xóa luôn các Job và Pod con của nó.

</p>
</details>

### Tạo một CronJob với image busybox chạy mỗi phút và ghi 'date; echo Hello from the Kubernetes cluster' ra standard output. CronJob phải bị hủy nếu nó mất quá 17 giây để bắt đầu chạy so với thời điểm đã lên lịch (tức là Job đã lỡ lịch chạy).

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create cronjob time-limited-job --image=busybox --restart=Never --dry-run=client --schedule="* * * * *" -o yaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > time-limited-job.yaml
vi time-limited-job.yaml
```
Thêm cronjob.spec.startingDeadlineSeconds=17

```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: time-limited-job
spec:
  startingDeadlineSeconds: 17 # thêm dòng này
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: time-limited-job
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
            image: busybox
            name: time-limited-job
            resources: {}
          restartPolicy: Never
  schedule: '* * * * *'
status: {}
```

> 💡 **Bổ sung:** Nói chính xác, `startingDeadlineSeconds` không "dừng" Job đang chạy, mà nếu controller không thể khởi tạo Job trong vòng 17 giây sau thời điểm lên lịch (ví dụ controller bị gián đoạn, hoặc `concurrencyPolicy: Forbid` đang chặn) thì lần chạy đó bị **bỏ qua** và tính là missed. Đây là trường của **CronJob** (`spec.startingDeadlineSeconds`), khác với `activeDeadlineSeconds` của **Job** ở bài tiếp theo.

</p>
</details>

### Tạo một CronJob với image busybox chạy mỗi phút và ghi 'date; echo Hello from the Kubernetes cluster' ra standard output. CronJob phải bị dừng nếu nó đã khởi chạy thành công nhưng mất quá 12 giây để chạy xong.

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create cronjob time-limited-job --image=busybox --restart=Never --dry-run=client --schedule="* * * * *" -o yaml -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster' > time-limited-job.yaml
vi time-limited-job.yaml
```
Thêm cronjob.spec.jobTemplate.spec.activeDeadlineSeconds=12

```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: time-limited-job
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: time-limited-job
    spec:
      activeDeadlineSeconds: 12 # thêm dòng này
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
            image: busybox
            name: time-limited-job
            resources: {}
          restartPolicy: Never
  schedule: '* * * * *'
status: {}
```

> 💡 **Bổ sung:** Cấu trúc lồng nhau của CronJob dễ nhầm khi thi: `CronJob.spec` → `jobTemplate.spec` (spec của **Job**: `activeDeadlineSeconds`, `backoffLimit`, `completions`...) → `template.spec` (spec của **Pod**: `containers`, `restartPolicy`...). Dùng `kubectl explain cronjob.spec.jobTemplate.spec` để kiểm tra nhanh trường nằm ở cấp nào.

</p>
</details>

### Chỉ giữ lại 2 lần chạy thành công và 1 lần chạy thất bại gần nhất của một CronJob

<details><summary>Xem lời giải</summary> <p>

Tạo một CronJob có cấu hình giới hạn lịch sử:
```bash
kubectl create cronjob history-demo \
  --image=busybox \
  --schedule="*/1 * * * *" \
  --dry-run=client -o yaml \
  -- /bin/sh -c 'date; echo Hello from history demo' > history-demo.yaml
```

Sửa file:

```bash
vi history-demo.yaml
```

Thêm các trường sau vào dưới spec:

```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  name: history-demo
spec:
  schedule: "*/1 * * * *"
  successfulJobsHistoryLimit: 2   # giữ lại 2 Job thành công gần nhất
  failedJobsHistoryLimit: 1        # giữ lại 1 Job thất bại gần nhất
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: history-demo
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from history demo
          restartPolicy: Never
```

Apply CronJob:
```bash
kubectl apply -f history-demo.yaml
```

Kiểm tra cách lịch sử Job được giữ lại:

```bash
kubectl get cj history-demo
kubectl get jobs --watch
```

Sau vài lần chạy, xác nhận rằng:
- Chỉ có 2 Job thành công được giữ lại
- Chỉ có 1 Job thất bại được giữ lại (nếu có lỗi xảy ra)

Dọn dẹp:
```bash
kubectl delete cj history-demo
```

> 💡 **Bổ sung:** Giá trị mặc định là `successfulJobsHistoryLimit: 3` và `failedJobsHistoryLimit: 1`. Đặt bằng `0` nghĩa là không giữ lại Job nào sau khi chạy xong (khi đó cũng mất luôn log của Pod).

</p> </details>

### Tạo một Job từ CronJob

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl create job --from=cronjob/sample-cron-job sample-job
```

> 💡 **Bổ sung:** Lệnh này tạo một Job từ `jobTemplate` của CronJob để chạy ngay lập tức, không cần chờ tới lịch. Cách này rất tiện để kiểm thử CronJob hoặc chạy bù một lần bị lỡ. Nếu muốn tạm ngưng lịch chạy của CronJob mà không xóa nó, dùng `kubectl patch cj sample-cron-job -p '{"spec":{"suspend":true}}'`.

</p>
</details>
