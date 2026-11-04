# Practice Test - Manual Scheduling (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-manual-scheduling/)

Lời giải cho bài Practice Test - Manual Scheduling

1.  <details>
    <summary>Cho sẵn file định nghĩa Pod nginx.yaml. Hãy tạo Pod từ file này.</summary>

    ```
    kubectl create -f nginx.yaml
    ```
    </details>

1.  <details>
    <summary>Trạng thái của Pod vừa tạo là gì?</summary>

    ```
    kubectl get pods
    ```

    Xem cột `STATUS`
    </details>

1.  <details>
    <summary>Tại sao Pod ở trạng thái Pending?</br>Kiểm tra các thành phần control plane của Kubernetes trong môi trường.</summary>

    ```
    kubectl get pods --namespace kube-system
    ```

    Thiếu một Pod quan trọng ở đây! (Không có `kube-scheduler`, nên không ai gán node cho Pod.)
    </details>

1.  <details>
    <summary>Lập lịch thủ công Pod lên node01.</summary>

    Ta phải xóa và tạo lại Pod, vì `nodeName` không thể sửa trên một Pod đang tồn tại (với Pod đang chạy, hầu như chỉ có thể sửa một số trường như `image`, `activeDeadlineSeconds`, `tolerations` (chỉ được thêm)).

    ```
    vi nginx.yaml
    ```

    Sửa file như sau

    ```yaml
    ---
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      nodeName: node01    # add this line
      containers:
      -  image: nginx
         name: nginx
    ```

    ```
    kubectl delete -f nginx.yaml
    kubectl create -f nginx.yaml
    ```

    > 💡 **Mẹo:** Có thể gộp hai lệnh trên thành `kubectl replace --force -f nginx.yaml`. Kiểm tra kết quả bằng `kubectl get pods -o wide` (cột `NODE`).
    </details>

1.  <details>
    <summary>Bây giờ hãy lập lịch chính Pod đó lên node controlplane.</summary>

    Lặp lại các bước như câu trước, sửa `nodeName` thành `controlplane`.

    Lưu ý: dù node controlplane có taint `node-role.kubernetes.io/control-plane:NoSchedule`, Pod vẫn chạy được vì `nodeName` bỏ qua scheduler (taint `NoSchedule` chỉ được scheduler xét đến).
  </details>

