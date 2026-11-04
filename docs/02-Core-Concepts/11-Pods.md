# Pods
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/pods-2/)
  
Trong phần này, chúng ta sẽ tìm hiểu về Pod.
- Giới thiệu Pod
- Làm sao để triển khai Pod?

#### Kubernetes không triển khai container trực tiếp lên worker node.

  ![pod](../../images/pod.PNG)

> 💡 **Bổ sung:** Container được đóng gói trong một đối tượng Kubernetes gọi là Pod. Pod là đơn vị nhỏ nhất bạn có thể tạo trong Kubernetes.
  
#### Đây là một cluster Kubernetes một node, với một instance của ứng dụng chạy trong một container Docker được đóng gói trong Pod.

![pod1](../../images/pod1.PNG)

#### Pod thường có quan hệ một-một với container chạy ứng dụng của bạn.

  ![pod2](../../images/pod2.PNG)

> 💡 **Bổ sung:** Khi cần scale ứng dụng để phục vụ nhiều người dùng hơn, bạn **tạo thêm Pod mới** (mỗi Pod chứa một instance ứng dụng), chứ không thêm container vào Pod hiện có. Khi cần giảm tải, bạn xóa bớt Pod. Việc này thường do ReplicaSet/Deployment đảm nhiệm thay vì làm thủ công.
  
## Pod nhiều container (Multi-Container Pods)
- Một Pod có thể chứa nhiều container, nhưng thường không phải là nhiều container **`same kind`** (cùng loại).
  
  ![pod3](../../images/pod3.PNG)

> 💡 **Bổ sung:** Các container trong cùng một Pod dùng chung network namespace (giao tiếp với nhau qua `localhost`), có thể dùng chung volume, và được tạo/xóa cùng nhau. Trường hợp điển hình là container "helper" đi kèm ứng dụng chính, ví dụ container thu thập log hoặc proxy (mẫu **sidecar**). Ngoài ra còn có **init container** — chạy tuần tự đến khi hoàn tất trước khi các container chính khởi động. Từ Kubernetes v1.29 (bật mặc định) có **native sidecar container**: khai báo trong `initContainers` với `restartPolicy: Always`, chạy song song suốt vòng đời Pod.
  
## Ví dụ với Docker (Docker Link)
  
  ![pod4](../../images/pod4.PNG)

> 💡 **Bổ sung:** Ví dụ này minh họa lý do Kubernetes dùng Pod: nếu chỉ chạy container thuần với Docker, bạn phải tự quản lý việc liên kết (`--link`) giữa container ứng dụng và container helper, tự tạo network, volume dùng chung và tự theo dõi vòng đời của chúng. Với Pod, Kubernetes tự động lo tất cả những việc đó.
  
## Làm sao để triển khai Pod?
Giờ hãy cùng tạo một Pod nginx bằng **`kubectl`**.

- Để triển khai một container Docker bằng cách tạo Pod.
  ```
  $ kubectl run nginx --image nginx
  ```

- Để lấy danh sách Pod
  ```
  $ kubectl get pods
  ```

 ![kubectl](../../images/kubectl.PNG)

> 💡 **Bổ sung — mẹo thi:**
> - Ở các bản Kubernetes hiện tại, `kubectl run` **chỉ tạo Pod** (tùy chọn `--generator` đã bị loại bỏ; muốn tạo Deployment hãy dùng `kubectl create deployment`).
> - Cách nhanh nhất để có file YAML mẫu: `kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml`, sửa file rồi `kubectl apply -f pod.yaml`.
> - Xem chi tiết Pod (node đang chạy, IP, events lỗi pull image...): `kubectl get pods -o wide` và `kubectl describe pod nginx`.
> - Pod không thể sửa hầu hết các trường khi đang chạy (trừ một số trường như `image`). Nếu cần thay đổi, dùng `kubectl get pod nginx -o yaml > pod.yaml`, sửa, rồi `kubectl replace --force -f pod.yaml`.

Tài liệu tham khảo K8s:
- https://kubernetes.io/docs/concepts/workloads/pods/pod/
- https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/
- https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro/

> Ghi chú: hai link đầu là đường dẫn cũ; trang tài liệu hiện tại về Pod nằm tại https://kubernetes.io/docs/concepts/workloads/pods/


