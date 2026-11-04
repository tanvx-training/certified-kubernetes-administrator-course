# Upgrade hệ điều hành (OS Upgrades)
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/os-upgrades/)
  
Trong phần này, chúng ta sẽ tìm hiểu cách upgrade hệ điều hành (hoặc bảo trì nói chung) trên các node.

#### Nếu node bị down quá 5 phút, các Pod trên node đó sẽ bị terminate

  ![os](../../images/os.PNG)

> 💡 **Bổ sung:** Khoảng thời gian này gọi là *pod eviction timeout*. Trước đây được cấu hình qua flag `--pod-eviction-timeout` của kube-controller-manager (mặc định 5 phút); flag này đã bị loại bỏ (từ v1.27). Ở Kubernetes hiện tại, cơ chế được thực hiện bằng **taint-based eviction**: khi node chuyển sang `NotReady`/`Unreachable`, node controller gắn taint `node.kubernetes.io/not-ready` hoặc `node.kubernetes.io/unreachable` (effect `NoExecute`), và mỗi Pod mặc định được thêm toleration với `tolerationSeconds: 300` (5 phút). Hết thời gian đó, Pod bị evict. Nếu Pod thuộc một ReplicaSet/Deployment thì sẽ được tạo lại trên node khác; nếu là Pod "trơ trọi" (không có controller quản lý) thì sẽ mất hẳn.
  
- Bạn có thể chủ động **`drain`** node để đẩy toàn bộ workload ra khỏi node đó, các workload sẽ được tạo lại trên những node khác.
  ```
  $ kubectl drain node-1
  ```
- Khi drain, node đồng thời bị cordon, tức là được đánh dấu unschedulable (không cho schedule Pod mới lên).
- Khi node hoạt động trở lại sau bảo trì, nó vẫn ở trạng thái unschedulable. Bạn cần uncordon node đó.
  ```
  $ kubectl uncordon node-1
  ```
- Ngoài ra còn có lệnh cordon. Cordon chỉ đơn giản đánh dấu node là unschedulable. Khác với drain, nó không terminate hay di chuyển các Pod đang chạy trên node.

  ![drain](../../images/drain.PNG)

> 💡 **Bổ sung:** Một số lưu ý thực tế khi drain:
> - Nếu node có Pod do DaemonSet quản lý, `kubectl drain` sẽ báo lỗi; cần thêm `--ignore-daemonsets` (Pod của DaemonSet sẽ bị bỏ qua, không bị xóa).
> - Nếu node có Pod không được quản lý bởi controller nào (ReplicaSet, Job, DaemonSet, StatefulSet...), drain sẽ từ chối; cần `--force`, và Pod đó sẽ **mất vĩnh viễn** vì không có gì tạo lại nó.
> - Nếu Pod dùng volume `emptyDir`, cần thêm `--delete-emptydir-data` (tên cũ `--delete-local-data` đã bị loại bỏ); dữ liệu trong `emptyDir` sẽ mất.
> - Drain tôn trọng PodDisruptionBudget (PDB): nếu việc evict vi phạm PDB, lệnh sẽ chờ/thử lại.
> - Uncordon không tự động chuyển Pod cũ về lại node; node chỉ nhận Pod khi có Pod mới được schedule.
> - Kiểm tra trạng thái: `kubectl get nodes` sẽ hiển thị `Ready,SchedulingDisabled` cho node bị cordon.
  
  
#### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/
