# Practice Test - ReplicaSets (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-replicasets/)

#### Lời giải cho bài practice test ReplicaSet
1. <details>
   <summary>Có bao nhiêu Pod đang tồn tại trên hệ thống?</summary>

   ```bash
   kubectl get pods
   ```

   Đếm số Pod hiển thị (nếu có).

   </details>

1. <details>
   <summary>Có bao nhiêu ReplicaSet đang tồn tại trên hệ thống?</summary>

   ```bash
   kubectl get replicasets
   ```

   Đếm số ReplicaSet hiển thị (nếu có).

   </details>


1. <details>
   <summary>Còn bây giờ thì sao? Bạn thấy bao nhiêu ReplicaSet?</summary>

   ```bash
   kubectl get replicasets
   ```

   Đếm số ReplicaSet hiển thị (nếu có).

   </details>

1. <details>
   <summary>new-replica-set mong muốn (DESIRED) bao nhiêu Pod?</summary>

   Từ output của câu 3, xem cột `DESIRED`.
   </details>

1. <details>
   <summary>Image nào được dùng để tạo các Pod trong new-replica-set?</summary>

   ```
   kubectl describe replicaset
   ```

   ...rồi xem trong mục containers --- hoặc --

   ```
   kubectl get rs -o wide
   ```

   ...rồi xem cột `IMAGES`. Kubernetes chấp nhận `rs` là tên viết tắt của `replicaset`.

   > 💡 **Bổ sung:** Có thể xem danh sách tên viết tắt của mọi loại resource bằng `kubectl api-resources` (cột `SHORTNAMES`), ví dụ `po` (pods), `deploy` (deployments), `svc` (services), `ns` (namespaces), `cm` (configmaps).

   </details>

1. <details>
   <summary>Có bao nhiêu Pod đang READY trong new-replica-set?</summary>

   ```
   kubectl get rs
   ```

   Xem cột `READY`.
   </details>

1. <details>
   <summary>Theo bạn, vì sao các Pod không ready?</summary>

   ```
   kubectl describe pods
   ```

   Xem mục `Events` ở cuối.
   </details>

1. <details>
   <summary>Xóa một Pod bất kỳ trong 4 Pod.</summary>

   ```
   kubectl get pods
   ```

   Chọn một Pod bất kỳ trong bốn Pod.

   ```
   kubectl delete pod new-replica-set-XXXX
   ```
   </details>

1. <details>
   <summary>Bây giờ có bao nhiêu Pod?</summary>

   ```
   kubectl get pods
   ```

   </details>

1. <details>
   <summary>Vì sao vẫn còn 4 Pod, dù bạn đã xóa một Pod?</summary>
   > ReplicaSet đảm bảo số lượng Pod mong muốn luôn được duy trì

   </details>

1. <details>
   <summary>Tạo một ReplicaSet bằng file replicaset-definition-1.yaml nằm ở /root/.</br>File này có lỗi, hãy thử sửa nó.</summary>

   ```
   kubectl create -f replicaset-definition-1.yaml
   ```

   Để ý thông báo lỗi.

   Lấy apiVersion của replicaset

   ```
   $ kubectl explain replicaset | grep VERSION
   ```

   Sửa file định nghĩa replicaset trong `vi` với version đúng rồi thử tạo lại.

   ```
   $ kubectl create -f replicaset-definition-1.yaml
   ```

   > 💡 **Bổ sung:** Lỗi ở đây là file dùng `apiVersion: v1`, trong khi ReplicaSet thuộc nhóm API `apps/v1` (lỗi kiểu `no matches for kind "ReplicaSet" in version "v1"`). `kubectl explain` cho biết `GROUP: apps` và `VERSION: v1`, nên giá trị đúng là `apps/v1`. Cũng có thể tra nhanh bằng `kubectl api-resources | grep -i replicaset`.

   </details>

1. <details>
   <summary>Sửa lỗi trong file replicaset-definition-2.yaml và tạo ReplicaSet từ file đó.</summary>

   ```
   kubectl create -f replicaset-definition-2.yaml
   ```

   Để ý thông báo lỗi.

   Selector matchLabels phải khớp với label của Pod - hãy sửa `replicaset-definition-2.yaml`.

   Giá trị label ở dòng 9 và dòng 13 phải giống nhau.

   ```
   $ kubectl create -f replicaset-definition-2.yaml
   ```
   </details>

1. <details>
   <summary>Xóa hai ReplicaSet vừa tạo - replicaset-1 và replicaset-2</summary>

   ```
   kubectl delete replicaset replicaset-1
   kubectl delete rs replicaset-2
   ```

   --- HOẶC ---

   ```
   kubectl delete replicaset replicaset-1 replicaset-2
   ```

   </details>

1. <details>
   <summary>Sửa ReplicaSet ban đầu new-replica-set để dùng đúng image busybox.</summary>

   ```
   kubectl edit replicaset new-replica-set
   ```

   Sửa image thành `busybox`, lưu và thoát.
   </details>

1. <details>
   <summary>Sửa ReplicaSet ban đầu new-replica-set để dùng đúng image busybox.</summary>

   ```
   kubectl edit replicaset new-replica-set
   ```

   Sửa image, lưu và thoát.

   Nếu chạy `kubectl get pods`, bạn sẽ thấy các Pod vẫn lỗi. ReplicaSet không "thông minh" lắm và không tự deploy lại Pod khi phần đặc tả container bị chỉnh sửa.

   > 💡 **Bổ sung:** ReplicaSet chỉ đảm bảo **số lượng** Pod khớp với selector, chứ không so sánh nội dung Pod với template. Template mới chỉ áp dụng cho các Pod được tạo **sau** khi sửa. Đây chính là lý do nên dùng Deployment: khi bạn đổi template, Deployment tạo ReplicaSet mới và thực hiện rolling update để thay thế toàn bộ Pod cũ.

   Ta phải xóa rồi tạo lại ReplicaSet bằng cách export YAML của nó...

   ```
   kubectl get rs new-replica-set -o yaml > rs.yaml
   kubectl delete rs new-replica-set
   kubectl create -f rs.yaml
   ```

   -- HOẶC --

   Xóa từng Pod bị lỗi. ReplicaSet sẽ tạo Pod mới thay thế, và Pod mới này sẽ chạy bình thường.

   -- HOẶC --

   Scale về 0, rồi scale lại lên 4
   ```
   kubectl scale rs new-replica-set --replicas 0
   kubectl scale rs new-replica-set --replicas 4
   ```

   > 💡 **Bổ sung:** Cách nhanh nhất để "xóa từng Pod lỗi" là xóa tất cả Pod theo label của ReplicaSet, ví dụ `kubectl delete pods -l name=busybox-pod` (thay bằng label thực tế, xem bằng `kubectl get rs new-replica-set -o wide` ở cột `SELECTOR`).

   </details>

1. <details>
   <summary>Scale ReplicaSet lên 5 Pod.</summary>

   ```
   kubectl scale rs new-replica-set --replicas 5
   ```

   </details>

