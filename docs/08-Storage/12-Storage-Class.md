# Storage Class

  - Xem [Bài giảng](https://kodekloud.com/topic/storage-class/)

Trong phần này, chúng ta sẽ tìm hiểu về **Storage Class**

- Chúng ta đã tìm hiểu cách tạo Persistent Volume, Persistent Volume Claim và cách dùng claim đó làm volume trong Pod.
- Ta đã tạo Persistent Volume, nhưng nếu volume đến từ các nhà cung cấp cloud như GCP, AWS, Azure thì trước đó ta phải tự tạo ổ đĩa trên cloud — ví dụ tạo disk trên Google Cloud.
- Mỗi lần ứng dụng cần lưu trữ, ta phải tự tay tạo disk trên cloud rồi tạo file định nghĩa PV tương ứng với đúng tên disk đó. Cách này gọi là **Static Provisioning** (cấp phát tĩnh).

#### Static Provisioning (cấp phát tĩnh)

![class-18](../../images/class18.PNG)


#### Dynamic Provisioning (cấp phát động)

![class-19](../../images/class19.PNG)

- Với Storage Class, ta định nghĩa một provisioner (vd Google Storage) có khả năng tự động cấp phát lưu trữ trên cloud và tự gắn vào Pod khi có claim. Nhờ vậy ta không cần định nghĩa Persistent Volume nữa: khi PVC tham chiếu tới StorageClass được tạo, provisioner sẽ tự tạo disk và PV tương ứng rồi bind với PVC. Cách này gọi là **Dynamic Provisioning** (cấp phát động).

```
sc-definition.yaml

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: google-storage
provisioner: kubernetes.io/gce-pd
```

#### Tạo Storage Class

```
$ kubectl create -f sc-definition.yaml
storageclass.storage.k8s.io/google-storage created
```

#### Liệt kê Storage Class

```
$ kubectl get sc
NAME             PROVISIONER            RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
google-storage   kubernetes.io/gce-pd   Delete          Immediate           false                  20s
```

> 💡 **Bổ sung — các trường quan trọng của StorageClass** (đều thấy được trong output `kubectl get sc`):
> - `provisioner`: driver dùng để tạo volume. `kubernetes.io/gce-pd` là provisioner in-tree kiểu cũ; trên cluster hiện đại hãy dùng CSI driver như `pd.csi.storage.gke.io` (GCP), `ebs.csi.aws.com` (AWS), `disk.csi.azure.com` (Azure). Với ổ lưu trữ cục bộ không hỗ trợ cấp phát động, dùng `kubernetes.io/no-provisioner` (PV vẫn phải tạo thủ công).
> - `reclaimPolicy`: `Delete` (mặc định) hoặc `Retain` — áp dụng cho các PV được StorageClass tạo ra.
> - `volumeBindingMode`:
>   - `Immediate` (mặc định): tạo và bind PV ngay khi PVC được tạo.
>   - `WaitForFirstConsumer`: trì hoãn việc tạo/bind PV cho đến khi có Pod sử dụng PVC được schedule; PVC sẽ ở trạng thái `Pending` cho tới lúc đó. Chế độ này giúp volume được tạo đúng zone/node nơi Pod chạy và là bắt buộc với `local` volume.
> - `allowVolumeExpansion: true`: cho phép tăng dung lượng PVC bằng cách sửa `spec.resources.requests.storage` (chỉ tăng, không giảm được).
> - `parameters`: tham số riêng của từng provisioner (vd `type: pd-ssd`, `replication-type: regional-pd` với GCE PD).
>
> **StorageClass mặc định:** đánh dấu bằng annotation `storageclass.kubernetes.io/is-default-class: "true"` (hiển thị `(default)` trong `kubectl get sc`). PVC không khai báo `storageClassName` sẽ tự dùng class mặc định. Ví dụ đầy đủ:
>
> ```yaml
> apiVersion: storage.k8s.io/v1
> kind: StorageClass
> metadata:
>   name: fast-ssd
>   annotations:
>     storageclass.kubernetes.io/is-default-class: "true"
> provisioner: pd.csi.storage.gke.io
> parameters:
>   type: pd-ssd
> reclaimPolicy: Retain
> volumeBindingMode: WaitForFirstConsumer
> allowVolumeExpansion: true
> ```

#### Tạo Persistent Volume Claim

```
pvc-definition.yaml

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes: [ "ReadWriteOnce" ]
  storageClassName: google-storage       
  resources:
   requests:
     storage: 500Mi
```
```
$ kubectl create -f pvc-definition.yaml

```
#### Tạo Pod

```
pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: frontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: web
  volumes:
    - name: web
      persistentVolumeClaim:
        claimName: myclaim
```
```
$ kubectl create -f pod-definition.yaml
```
- Khi PVC trên được tạo, StorageClass sẽ dùng provisioner để tạo một disk mới có dung lượng như yêu cầu trên GCP, sau đó tạo PV và bind PV đó với PVC. Như vậy PV vẫn được tạo, chỉ là ta không phải tự tạo thủ công nữa.

#### Provisioner

![class-20](../../images/class20.PNG)

- Ngoài GCE, còn nhiều provisioner khác như AWS EBS, Azure File, Azure Disk, CephFS, Portworx... Mỗi provisioner có thể nhận thêm các tham số (`parameters`) riêng, như loại disk, kiểu replication. Từ đó ta có thể tạo nhiều StorageClass với các mức dịch vụ khác nhau (vd `silver`, `gold`, `platinum`).

#### Tài liệu tham khảo Kubernetes Storage Class

- https://kubernetes.io/docs/concepts/storage/storage-classes/
- https://cloud.google.com/kubernetes-engine/docs/concepts/persistent-volumes#storageclasses
- https://docs.aws.amazon.com/eks/latest/userguide/storage-classes.html
