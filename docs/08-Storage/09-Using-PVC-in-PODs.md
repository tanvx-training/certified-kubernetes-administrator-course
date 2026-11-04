# Sử dụng PVC trong Pod

  - Xem [Bài giảng](https://kodekloud.com/topic/using-pvc-in-pods/)

Trong phần này, chúng ta sẽ tìm hiểu cách **sử dụng PVC trong Pod**

- Pod truy cập lưu trữ bằng cách dùng claim như một volume. Persistent Volume Claim phải nằm trong **cùng namespace** với Pod sử dụng claim đó.
- Cluster tìm claim trong namespace của Pod và dùng nó để lấy Persistent Volume đứng sau claim. Sau đó volume được mount lên host và vào bên trong Pod.
- Persistent Volume là tài nguyên cấp cluster (cluster-scoped), còn Persistent Volume Claim là tài nguyên cấp namespace (namespace-scoped).

#### Tạo Persistent Volume

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

```

#### Tạo Persistent Volume Claim

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
    - name: myfrontend
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

#### Liệt kê Pod, Persistent Volume và Persistent Volume Claim

```
$ kubectl get pod,pvc,pv

```

> 💡 **Bổ sung:** Với Deployment hay ReplicaSet, cách dùng PVC cũng tương tự: khai báo `persistentVolumeClaim.claimName` trong phần `volumes` của Pod template. Lưu ý với Deployment nhiều replica và PVC kiểu `ReadWriteOnce`, các Pod nằm trên node khác sẽ không mount được volume; StatefulSet dùng `volumeClaimTemplates` để tạo PVC riêng cho từng Pod. Nếu Pod ở trạng thái `Pending`/`ContainerCreating` vì volume, hãy kiểm tra `kubectl describe pod <tên>` và trạng thái PVC (phải là `Bound`).

#### Tài liệu tham khảo

- https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes



