# State Persistence - Lưu trữ dữ liệu bền vững (8%)

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Pod to Use a Volume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/)

kubernetes.io > Documentation > Tasks > Configure Pods and Containers > [Configure a Pod to Use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

## Định nghĩa volume

### Tạo Pod busybox có hai container, cả hai dùng image busybox và chạy lệnh 'sleep 3600'. Cho cả hai container mount một emptyDir tại '/etc/foo'. Vào container busybox thứ hai, ghi cột đầu tiên của file '/etc/passwd' vào '/etc/foo/passwd'. Vào container busybox thứ nhất và in file '/etc/foo/passwd' ra standard output. Xóa Pod.

<details><summary>Xem lời giải</summary>
<p>

*Câu này có lẽ hợp với phần 'Multi-container-pods' hơn, nhưng tác giả để ở đây vì nó giúp bạn làm quen với khái niệm state (trạng thái/dữ liệu)*

Cách dễ nhất là tạo một Pod mẫu bằng lệnh:

```bash
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run=client -- /bin/sh -c 'sleep 3600' > pod.yaml
vi pod.yaml
```
Copy rồi paste phần định nghĩa container, sau đó gõ thêm các dòng có comment ở cuối:

```YAML
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    name: busybox2 # nhớ đổi tên khi copy paste, phải khác tên container thứ nhất!
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  volumes: #
  - name: myvolume #
    emptyDir: {} #
```
Nếu bạn quên thêm ```bash -- /bin/sh -c 'sleep 3600'``` vào lệnh tạo Pod mẫu, có thể khai báo trường command trong file cấu hình:

```YAML
spec:
  containers:
  - image: busybox
    name: busybox
    command: ["/bin/sh", "-c", "sleep 3600"]
```

Vào container thứ hai:

```bash
kubectl exec -it busybox -c busybox2 -- /bin/sh
cat /etc/passwd | cut -f 1 -d ':' > /etc/foo/passwd # thay vì cut có thể dùng awk -F ":" '{print $1}'
cat /etc/foo/passwd # xác nhận nội dung đã được ghi thành công
exit
```

Vào container thứ nhất:

```bash
kubectl exec -it busybox -c busybox -- /bin/sh
mount | grep foo # xác nhận volume đã được mount
cat /etc/foo/passwd
exit
kubectl delete po busybox
```

> 💡 **Bổ sung:**
> - `emptyDir` được tạo khi Pod được gán vào node và **bị xóa vĩnh viễn khi Pod bị xóa**. Container restart không làm mất dữ liệu, xóa Pod thì mất. Volume này dùng để chia sẻ file giữa các container trong cùng Pod (sidecar, init container), không dùng để lưu trữ bền vững.
> - Có thể đặt `emptyDir: {medium: Memory, sizeLimit: 64Mi}` để dùng tmpfs (RAM).
> - Không cần `exec` vào shell, chạy một lệnh là đủ: `kubectl exec busybox -c busybox -- cat /etc/foo/passwd`.

</p>
</details>


### Tạo một PersistentVolume 10Gi tên 'myvolume'. Cho nó accessMode 'ReadWriteOnce' và 'ReadWriteMany', storageClassName 'normal', mount trên hostPath '/etc/foo'. Lưu vào pv.yaml rồi tạo trên cluster. Hiển thị các PersistentVolume đang có trên cluster

<details><summary>Xem lời giải</summary>
<p>

```bash
vi pv.yaml
```

```YAML
kind: PersistentVolume
apiVersion: v1
metadata:
  name: myvolume
spec:
  storageClassName: normal
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
    - ReadWriteMany
  hostPath:
    path: /etc/foo
```

Hiển thị các PersistentVolume:

```bash
kubectl create -f pv.yaml
# sẽ có trạng thái 'Available'
kubectl get pv
```

> 💡 **Bổ sung:**
> - Không có lệnh `kubectl create pv` imperative, nên khi thi hãy copy YAML mẫu từ trang tài liệu (kubernetes.io > Documentation > Concepts > Storage > Persistent Volumes) rồi sửa.
> - PV là resource **cấp cluster** (không thuộc namespace nào), còn PVC thuộc namespace.
> - Các access mode: `ReadWriteOnce` (RWO, mount đọc-ghi bởi một node), `ReadOnlyMany` (ROX), `ReadWriteMany` (RWX), `ReadWriteOncePod` (RWOP, chỉ một Pod duy nhất, GA từ v1.29). Bản thân `hostPath` không thật sự hỗ trợ RWX giữa nhiều node; khai báo ở đây chỉ để luyện tập.
> - `persistentVolumeReclaimPolicy` mặc định là `Retain` với PV tạo thủ công. Khi xóa PVC, PV chuyển sang `Released` và không được bind lại cho đến khi admin dọn dẹp.

</p>
</details>

### Tạo PersistentVolumeClaim tên 'mypvc' cho PersistentVolume này, request 4Gi, accessMode ReadWriteOnce, storageClassName normal, lưu vào pvc.yaml. Tạo nó trên cluster. Hiển thị các PersistentVolumeClaim và các PersistentVolume của cluster

<details><summary>Xem lời giải</summary>
<p>

```bash
vi pvc.yaml
```

```YAML
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mypvc
spec:
  storageClassName: normal
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 4Gi
```

Tạo trên cluster:

```bash
kubectl create -f pvc.yaml
```

Hiển thị PersistentVolumeClaim và PersistentVolume:

```bash
kubectl get pvc # sẽ hiện 'Bound'
kubectl get pv # cũng sẽ hiện 'Bound'
```

> 💡 **Bổ sung:**
> - Một PVC bind được vào PV khi **khớp `storageClassName`**, access mode của PVC là tập con của PV, và dung lượng PV ≥ dung lượng request. Quan hệ bind là 1-1: PVC 4Gi bind vào PV 10Gi sẽ chiếm trọn PV đó, và `kubectl get pvc` sẽ báo `CAPACITY` là 10Gi.
> - PVC kẹt ở `Pending` thường do sai `storageClassName`, sai access mode, hoặc không có PV nào đủ lớn. Xem lý do bằng `kubectl describe pvc mypvc` (phần Events).
> - Nếu StorageClass có `volumeBindingMode: WaitForFirstConsumer` (phổ biến với local-path, các CSI driver trên cloud), PVC sẽ `Pending` cho tới khi có Pod dùng nó. Đây là hành vi bình thường, không phải lỗi.
> - Nếu không ghi `storageClassName`, PVC sẽ dùng StorageClass mặc định của cluster (nếu có) và đi theo dynamic provisioning. Muốn bind tĩnh vào PV không có class thì đặt `storageClassName: ""`. PV/PVC/StorageClass nằm trong phần Storage của curriculum CKA.

</p>
</details>

### Tạo Pod busybox chạy lệnh 'sleep 3600', lưu vào pod.yaml. Mount PersistentVolumeClaim vào '/etc/foo'. Vào Pod 'busybox' và copy file '/etc/passwd' sang '/etc/foo/passwd'

<details><summary>Xem lời giải</summary>
<p>

Tạo khung Pod:

```bash
kubectl run busybox --image=busybox --restart=Never -o yaml --dry-run=client -- /bin/sh -c 'sleep 3600' > pod.yaml
vi pod.yaml
```

Thêm các dòng có comment ở cuối:

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
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    imagePullPolicy: IfNotPresent
    name: busybox
    resources: {}
    volumeMounts: #
    - name: myvolume #
      mountPath: /etc/foo #
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  volumes: #
  - name: myvolume #
    persistentVolumeClaim: #
      claimName: mypvc #
status: {}
```

Tạo Pod:

```bash
kubectl create -f pod.yaml
```

Vào Pod và copy '/etc/passwd' sang '/etc/foo/passwd':

```bash
kubectl exec busybox -it -- cp /etc/passwd /etc/foo/passwd
```

</p>
</details>

### Tạo Pod thứ hai giống hệt Pod vừa tạo (dễ nhất là đổi thuộc tính 'name' trong pod.yaml). Vào Pod đó và kiểm tra '/etc/foo' có chứa file 'passwd'. Xóa các Pod để dọn dẹp. Lưu ý: nếu không thấy file từ Pod thứ hai, bạn có đoán được tại sao không? Bạn sẽ khắc phục thế nào?



<details><summary>Xem lời giải</summary>
<p>

Tạo Pod thứ hai tên busybox2:

```bash
vim pod.yaml
# đổi 'metadata.name: busybox' thành 'metadata.name: busybox2'
kubectl create -f pod.yaml
kubectl exec busybox2 -- ls /etc/foo # sẽ hiện 'passwd'
# dọn dẹp
kubectl delete po busybox busybox2
kubectl delete pvc mypvc
kubectl delete pv myvolume
```

Nếu file hiện ở Pod thứ nhất mà không hiện ở Pod thứ hai, nhiều khả năng Pod thứ hai đã được schedule lên một node khác.

```bash
# kiểm tra các Pod đang nằm trên node nào
kubectl get po busybox -o wide
kubectl get po busybox2 -o wide
```

Nếu chúng nằm trên các node khác nhau thì bạn sẽ không thấy file, vì chúng ta dùng loại volume `hostPath`.
Nếu cần truy cập cùng một bộ file trên cluster nhiều node, bạn cần loại volume không phụ thuộc vào một node cụ thể.
Mỗi cloud provider có nhiều loại khác nhau [(xem tại đây)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes); giải pháp chung có thể là dùng NFS.

> 💡 **Bổ sung:** Một PVC RWO vẫn được nhiều Pod dùng chung nếu các Pod đó nằm **cùng một node** (RWO giới hạn theo node, không theo Pod). Muốn chỉ một Pod duy nhất được dùng thì chọn `ReadWriteOncePod`. Với bài lab dùng `hostPath`, có thể ép hai Pod lên cùng node bằng `nodeName` hoặc `nodeSelector`. Thứ tự xóa cũng đáng nhớ: nhờ cơ chế bảo vệ (finalizer `kubernetes.io/pvc-protection`), PVC đang được Pod dùng sẽ kẹt ở `Terminating` cho tới khi Pod bị xóa.

</p>
</details>

### Tạo Pod busybox với 'sleep 3600' làm argument. Copy '/etc/passwd' từ Pod về thư mục local của bạn

<details><summary>Xem lời giải</summary>
<p>

```bash
kubectl run busybox --image=busybox --restart=Never -- sleep 3600
kubectl cp busybox:/etc/passwd ./passwd # lệnh kubectl cp
# lệnh trên có thể báo lỗi, cứ bỏ qua vì việc copy vẫn thành công
cat passwd
```

> 💡 **Bổ sung:** `kubectl cp` chạy `tar` bên trong container, nên image **bắt buộc phải có binary `tar`** (image distroless/scratch sẽ báo lỗi). Thông báo hay gặp là `tar: removing leading '/' from member names`, đây chỉ là cảnh báo. Pod có nhiều container thì thêm `-c <container>`; Pod ở namespace khác thì viết `<namespace>/<pod>:/path`.

</p>
</details>
