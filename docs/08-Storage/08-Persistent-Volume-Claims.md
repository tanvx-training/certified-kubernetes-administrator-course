# Persistent Volume Claims

  - Xem [Bài giảng](https://kodekloud.com/topic/persistent-volume-claims-4/)

Trong phần này, chúng ta sẽ tìm hiểu về **Persistent Volume Claim**

- Bây giờ ta sẽ tạo một Persistent Volume Claim (PVC) để đưa phần lưu trữ đến với Pod sử dụng nó.
- Persistent Volume và Persistent Volume Claim là hai object riêng biệt trong Kubernetes. Quản trị viên tạo tập các Persistent Volume, còn người dùng tạo Persistent Volume Claim để sử dụng lưu trữ. (Lưu ý: PV là tài nguyên cấp cluster, còn PVC thuộc về một namespace.)
- Khi Persistent Volume Claim được tạo, Kubernetes sẽ bind (gắn) Persistent Volume với claim dựa trên yêu cầu (request) và các thuộc tính được thiết lập trên volume.


![class-17](../../images/class17.PNG)

- Nếu các thuộc tính không khớp hoặc không có Persistent Volume nào khả dụng cho Persistent Volume Claim, PVC sẽ ở trạng thái `Pending`.

> 💡 **Bổ sung — Cơ chế binding:** Mỗi PVC chỉ bind với **đúng một** PV và quan hệ này là 1-1: một PV đã bind sẽ không thể dùng cho PVC khác, kể cả khi PV còn dư dung lượng (vd PVC xin 500Mi bind vào PV 1Gi thì phần còn lại cũng không ai dùng được). Kubernetes chọn PV thỏa mãn tất cả các tiêu chí:
> - **Capacity**: dung lượng PV ≥ dung lượng PVC yêu cầu.
> - **Access Modes**: PV phải hỗ trợ access mode mà PVC yêu cầu.
> - **StorageClass**: `storageClassName` của PV và PVC phải giống nhau. Lưu ý: nếu PVC không khai báo `storageClassName` mà cluster có StorageClass mặc định, PVC sẽ tự nhận StorageClass mặc định đó và có thể không bind được với PV tạo thủ công không có StorageClass. Khi đó đặt `storageClassName: ""` trong PVC để bind với PV không có class.
> - **Volume Mode** (`Filesystem`/`Block`) phải khớp.
> - **Selector** (tùy chọn): PVC có thể dùng `selector.matchLabels` để chỉ bind với PV mang label cụ thể.
>
> Trong các PV phù hợp, Kubernetes ưu tiên PV có dung lượng nhỏ nhất đáp ứng được yêu cầu. Khi PVC đang `Pending`, dùng `kubectl describe pvc <tên>` để xem phần Events cho biết lý do.

```
pvc-definition.yaml

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes: [ "ReadWriteOnce" ]
  resources:
   requests:
     storage: 1Gi
```

```
pv-definition.yaml

kind: PersistentVolume
apiVersion: v1
metadata:
    name: pv-vol1
spec:
    accessModes: [ "ReadWriteOnce" ]
    capacity:
     storage: 1Gi
    hostPath:
     path: /tmp/data
```

#### Tạo Persistent Volume

```
$ kubectl create -f pv-definition.yaml
persistentvolume/pv-vol1 created

$ kubectl get pv
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv-vol1   1Gi        RWO            Retain           Available                                   10s
```


#### Tạo Persistent Volume Claim

```
$ kubectl create -f pvc-definition.yaml
persistentvolumeclaim/myclaim created

$ kubectl get pvc
NAME      STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
myclaim   Pending                                                     35s

$ kubectl get pvc
NAME      STATUS   VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
myclaim   Bound    pv-vol1   1Gi        RWO                           1min

```

#### Xóa Persistent Volume Claim

```
$ kubectl delete pvc myclaim
```

> 💡 **Bổ sung:** Điều gì xảy ra với PV sau khi xóa PVC phụ thuộc vào `persistentVolumeReclaimPolicy` của PV: với `Retain` (mặc định cho PV tạo thủ công), PV chuyển sang `Released` và giữ nguyên dữ liệu nhưng không thể bind lại với PVC mới; với `Delete`, PV và tài nguyên lưu trữ bên dưới bị xóa. Ngoài ra, nếu PVC đang được một Pod sử dụng, lệnh xóa sẽ khiến PVC kẹt ở trạng thái `Terminating` cho đến khi Pod bị xóa (cơ chế bảo vệ `kubernetes.io/pvc-protection`).

#### Xóa Persistent Volume

```
$ kubectl delete pv pv-vol1
```


#### Tài liệu tham khảo Kubernetes Persistent Volume Claims

- https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims
- https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#persistentvolumeclaim-v1-core
- https://docs.cloud.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengcreatingpersistentvolumeclaim.htm
