# Volumes

  - Xem [Bài giảng](https://kodekloud.com/topic/volumes/)

Trong phần này, chúng ta sẽ tìm hiểu về **Volumes**

- Như đã tìm hiểu ở phần Docker storage, nếu không gắn volume cho container thì khi container bị xóa, toàn bộ dữ liệu sẽ mất. Vì vậy, để lưu trữ dữ liệu bền vững, ta gắn volume vào container khi tạo chúng.
- Dữ liệu do container xử lý lúc này được đặt trong volume, nhờ đó được giữ lại lâu dài. Ngay cả khi container bị xóa, dữ liệu vẫn còn trong volume.

- Trong thế giới Kubernetes, các Pod có bản chất tạm thời (transient). Khi một Pod được tạo ra để xử lý dữ liệu rồi bị xóa, dữ liệu nó đã xử lý cũng bị xóa theo.
- Ví dụ, ta tạo một Pod đơn giản sinh ra một số ngẫu nhiên từ 1 đến 100 và ghi vào file `/opt/number.out`. Ta muốn lưu kết quả này vào volume để không bị mất.
- Ta tạo một volume cho việc đó. Trong ví dụ này, ta chỉ định đường dẫn `/data` trên host, nghĩa là các file được lưu trong thư mục `/data` trên node. Ta dùng trường `volumeMounts` trong mỗi container để mount `data-volume` vào thư mục `/opt` bên trong container. Số ngẫu nhiên giờ sẽ được ghi vào `/opt` bên trong container, mà thực chất nằm trên `data-volume`, tức thư mục `/data` trên host. Khi Pod bị xóa, file chứa số ngẫu nhiên vẫn còn trên host.

![class-14](../../images/class14.PNG)

> 💡 **Bổ sung:** Manifest tương ứng với ví dụ trên (dạng `hostPath`):
>
> ```yaml
> apiVersion: v1
> kind: Pod
> metadata:
>   name: random-number-generator
> spec:
>   containers:
>   - name: alpine
>     image: alpine
>     command: ["/bin/sh", "-c"]
>     args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
>     volumeMounts:
>     - mountPath: /opt
>       name: data-volume
>   volumes:
>   - name: data-volume
>     hostPath:
>       path: /data
>       type: DirectoryOrCreate
> ```
>
> Trường `hostPath.type` (tùy chọn) có các giá trị hay gặp: `Directory` (thư mục phải tồn tại sẵn), `DirectoryOrCreate` (tự tạo nếu chưa có), `File`, `FileOrCreate`... Ngoài `hostPath`, một loại volume rất hay gặp khi thi là `emptyDir`: thư mục rỗng được tạo khi Pod chạy trên node và **bị xóa khi Pod bị xóa** — thường dùng để chia sẻ dữ liệu giữa các container trong cùng một Pod (vd sidecar đọc log).



## Các lựa chọn lưu trữ cho Volume

- Volume kiểu `hostPath` phù hợp khi chỉ có một node. Không nên dùng nó cho cluster nhiều node, vì mỗi node có thư mục `/data` riêng và dữ liệu giữa các node không giống nhau (Pod chạy lại trên node khác sẽ không thấy dữ liệu cũ).
- Kubernetes hỗ trợ nhiều giải pháp lưu trữ chuẩn như NFS, GlusterFS, CephFS hoặc các giải pháp public cloud như AWS EBS, Azure Disk hay Google Persistent Disk.

![class-15](../../images/class15.PNG)



```
volumes:
- name: data-volume
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```

> 💡 **Bổ sung:** Ví dụ `awsElasticBlockStore` ở trên là volume plugin in-tree kiểu cũ; nó đã bị deprecated từ v1.19 và **bị gỡ bỏ hoàn toàn từ Kubernetes v1.27** (tương tự `gcePersistentDisk`, `azureDisk`...). Trên cluster hiện đại, hãy dùng CSI driver tương ứng, thường thông qua PersistentVolume/PersistentVolumeClaim và StorageClass (các phần tiếp theo) thay vì khai báo trực tiếp trong Pod.

#### Tài liệu tham khảo Kubernetes Volumes

- https://kubernetes.io/docs/concepts/storage/volumes/
- https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/
- https://unofficial-kubernetes.readthedocs.io/en/latest/concepts/storage/volumes/
- https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#volume-v1-core
