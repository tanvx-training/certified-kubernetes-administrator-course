# Cấu hình Kubernetes Scheduler
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/configuring-kubernetes-scheduler/)
  
Trong phần này, chúng ta sẽ tìm hiểu cách cấu hình Kubernetes scheduler (scheduler profiles).

![ks](../../images/ks.PNG)

> 💡 **Bổ sung — các giai đoạn lập lịch (scheduling framework):**
>
> Khi một Pod được tạo, nó đi qua các giai đoạn sau trước khi được gán vào node:
> 1. **Scheduling Queue**: Pod chờ trong hàng đợi, được sắp xếp theo độ ưu tiên (priority). Plugin: `PrioritySort`.
> 2. **Filtering**: loại bỏ những node không thể chạy Pod (thiếu tài nguyên, có taint không tolerate, không khớp node affinity, không khớp `nodeName`, node `unschedulable`...). Plugin: `NodeResourcesFit`, `NodeName`, `NodeUnschedulable`, `TaintToleration`, `NodeAffinity`, `NodePorts`...
> 3. **Scoring**: chấm điểm các node còn lại, ví dụ node còn nhiều tài nguyên trống hơn sau khi đặt Pod, hay node đã có sẵn image (`ImageLocality`) sẽ được điểm cao hơn. Plugin: `NodeResourcesFit`, `ImageLocality`, `TaintToleration`, `NodeAffinity`...
> 4. **Binding**: gán Pod vào node có điểm cao nhất bằng cách tạo Binding. Plugin: `DefaultBinder`.
>
> Mỗi plugin gắn vào một hoặc nhiều **extension point** (điểm mở rộng): `queueSort`, `preFilter`, `filter`, `postFilter`, `preScore`, `score`, `reserve`, `permit`, `preBind`, `bind`, `postBind`. Bạn có thể tự viết plugin và gắn vào các điểm này.

> 💡 **Bổ sung — Scheduler profiles:**
>
> Trước đây, muốn có scheduler với hành vi khác thì phải chạy thêm một tiến trình scheduler riêng (xem phần Multiple Schedulers). Cách này có nhược điểm: phải bảo trì nhiều tiến trình, và các scheduler riêng biệt có thể xung đột (race condition) khi cùng đặt Pod lên một node vì chúng không biết về nhau.
>
> Từ v1.18, một kube-scheduler có thể có nhiều **profile**. Mỗi profile có `schedulerName` riêng và bật/tắt plugin riêng, nhưng tất cả chạy trong cùng một tiến trình:
> ```
> apiVersion: kubescheduler.config.k8s.io/v1
> kind: KubeSchedulerConfiguration
> profiles:
> - schedulerName: my-scheduler-2
>   plugins:
>     score:
>       disabled:
>       - name: TaintToleration
>       enabled:
>       - name: MyCustomPluginA
>       - name: MyCustomPluginB
> - schedulerName: my-scheduler-3
>   plugins:
>     preScore:
>       disabled:
>       - name: '*'
>     score:
>       disabled:
>       - name: '*'
> - schedulerName: my-scheduler-4
> ```
> Pod chọn profile bằng `spec.schedulerName`, giống như với scheduler riêng biệt. Pod không khai báo `schedulerName` sẽ dùng profile `default-scheduler`.

> 💡 **Bổ sung — PriorityClass và preemption:**
>
> Thứ tự trong scheduling queue được quyết định bởi priority của Pod, được khai báo qua đối tượng `PriorityClass` (không thuộc namespace nào):
> ```
> apiVersion: scheduling.k8s.io/v1
> kind: PriorityClass
> metadata:
>   name: high-priority
> value: 1000000
> globalDefault: false
> preemptionPolicy: PreemptLowerPriority
> description: "Dùng cho các Pod quan trọng"
> ```
> ```
> apiVersion: v1
> kind: Pod
> metadata:
>   name: nginx
> spec:
>   priorityClassName: high-priority
>   containers:
>   - name: nginx
>     image: nginx
> ```
> - Tạo nhanh bằng lệnh imperative: `kubectl create priorityclass high-priority --value=1000000 --description="Dùng cho các Pod quan trọng"`. Xem danh sách: `kubectl get priorityclass`.
> - `value` càng lớn, priority càng cao. Giá trị do người dùng đặt tối đa là 1.000.000.000; các giá trị cao hơn dành cho hai PriorityClass hệ thống có sẵn là `system-cluster-critical` và `system-node-critical` (dùng cho các thành phần quan trọng như CoreDNS, kube-proxy...).
> - Pod không có `priorityClassName` có priority `0`, trừ khi có một PriorityClass đặt `globalDefault: true` (chỉ được có một).
> - **Preemption**: nếu Pod priority cao không tìm được node, scheduler có thể evict các Pod priority thấp hơn trên một node để nhường chỗ. Đặt `preemptionPolicy: Never` để Pod được xếp lên đầu hàng đợi nhưng không đuổi Pod khác.
> - Kiểm tra priority của Pod: `kubectl get pod <tên> -o jsonpath='{.spec.priority}'`.

## Tài liệu tham khảo
- https://github.com/kubernetes/community/blob/master/contributors/devel/sig-scheduling/scheduler.md
- https://kubernetes.io/blog/2017/03/advanced-scheduling-in-kubernetes/
- https://jvns.ca/blog/2017/07/27/how-does-the-kubernetes-scheduler-work/
- https://stackoverflow.com/questions/28857993/how-does-kubernetes-scheduler-work
- https://kubernetes.io/docs/reference/scheduling/config/
- https://kubernetes.io/docs/concepts/scheduling-eviction/scheduling-framework/
- https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/

