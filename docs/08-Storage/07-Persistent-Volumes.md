# Persistent Volumes

  - Xem [Bài giảng](https://kodekloud.com/topic/persistent-volumes-4/)

Trong phần này, chúng ta sẽ tìm hiểu về **Persistent Volume**

- Trong một môi trường lớn, có nhiều người dùng triển khai nhiều Pod, mỗi lần tạo Pod người dùng lại phải tự cấu hình lưu trữ cho từng Pod.
- Dù dùng giải pháp lưu trữ nào, người triển khai Pod cũng phải cấu hình nó trong tất cả các file định nghĩa Pod của mình. Mỗi khi cần thay đổi, người dùng lại phải sửa trên toàn bộ các Pod.

![class-16](../../images/class16.PNG)


- Persistent Volume (PV) là một "kho" (pool) volume lưu trữ ở cấp cluster, được quản trị viên cấu hình sẵn để những người triển khai ứng dụng trên cluster sử dụng. Người dùng giờ đây có thể lấy lưu trữ từ kho này thông qua Persistent Volume Claim (PVC).

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

  ```
  $ kubectl create -f pv-definition.yaml
  persistentvolume/pv-vol1 created

  $ kubectl get pv
  NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
  pv-vol1   1Gi        RWO            Retain           Available                                   3min
  
  $ kubectl delete pv pv-vol1
  persistentvolume "pv-vol1" deleted
  ```

> 💡 **Bổ sung — Access Modes:** Trường `accessModes` xác định cách volume có thể được mount:
> - `ReadWriteOnce` (**RWO**): mount đọc-ghi bởi **một node** (nhiều Pod trên cùng node đó vẫn dùng được).
> - `ReadOnlyMany` (**ROX**): mount chỉ đọc bởi nhiều node.
> - `ReadWriteMany` (**RWX**): mount đọc-ghi bởi nhiều node (vd NFS, CephFS).
> - `ReadWriteOncePod` (**RWOP**): mount đọc-ghi bởi **duy nhất một Pod** trong toàn cluster (GA từ v1.29, chỉ hỗ trợ với CSI volume).
>
> Không phải loại lưu trữ nào cũng hỗ trợ mọi access mode (vd AWS EBS chỉ hỗ trợ RWO).
>
> 💡 **Bổ sung — Reclaim Policy:** Trường `persistentVolumeReclaimPolicy` quyết định điều gì xảy ra với PV khi PVC gắn với nó bị xóa:
> - `Retain`: giữ lại PV và dữ liệu; PV chuyển sang trạng thái `Released` và **không thể** bind với PVC khác cho đến khi quản trị viên xử lý thủ công. Đây là mặc định với PV tạo thủ công (static).
> - `Delete`: xóa PV cùng tài nguyên lưu trữ bên dưới (vd ổ đĩa cloud). Đây là mặc định với PV được StorageClass tạo động.
> - `Recycle`: xóa sạch dữ liệu (`rm -rf /thevolume/*`) rồi cho PV sẵn sàng trở lại — **đã deprecated**, nên dùng dynamic provisioning thay thế.
>
> Các trạng thái (STATUS) của PV: `Available` → `Bound` → `Released` (hoặc `Failed`). Ví dụ đầy đủ hơn:
>
> ```yaml
> apiVersion: v1
> kind: PersistentVolume
> metadata:
>   name: pv-vol1
> spec:
>   capacity:
>     storage: 1Gi
>   accessModes:
>   - ReadWriteOnce
>   persistentVolumeReclaimPolicy: Retain
>   hostPath:
>     path: /tmp/data
> ```
>
> Mẹo thi: không có lệnh `kubectl create pv` — hãy copy mẫu YAML từ trang tài liệu Persistent Volumes trên kubernetes.io rồi sửa lại.

#### Tài liệu tham khảo Kubernetes Persistent Volumes

- https://kubernetes.io/docs/concepts/storage/persistent-volumes/
- https://portworx.com/tutorial-kubernetes-persistent-volumes/

