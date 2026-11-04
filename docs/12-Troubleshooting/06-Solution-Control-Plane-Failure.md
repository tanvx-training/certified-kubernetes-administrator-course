# Lời giải Control Plane Failure

  - Cùng xem lời giải cho [Practice Test](https://kodekloud.com/topic/practice-test-control-plane-failure/) về Control Plane Failure

    ### Lời giải

    1. Xem lời giải 

       <details>

        ```
        kubectl get pods -n kube-system
        ```

        ```
        sed -i 's/kube-schedulerrrr/kube-scheduler/g' /etc/kubernetes/manifests/kube-scheduler.yaml
        ```

        > 💡 **Giải thích:** Pod `kube-scheduler-controlplane` bị lỗi (`CrashLoopBackOff`/`RunContainerError`). `kubectl describe pod kube-scheduler-controlplane -n kube-system` cho thấy lệnh khởi động sai tên binary (`kube-schedulerrrr`). Vì scheduler là static Pod, ta sửa trực tiếp file manifest trong `/etc/kubernetes/manifests/`; kubelet sẽ tự tạo lại Pod. Khi scheduler hỏng, các Pod mới sẽ nằm ở trạng thái `Pending` vì không được gán node.
       </details>

    2. Xem lời giải

       <details>

        ```
        kubectl scale deploy app --replicas=2
        ```

        > 💡 **Giải thích:** Sau khi scale, nếu số Pod không tăng lên thì nghĩa là kube-controller-manager (thành phần chạy ReplicaSet controller) đang có vấn đề — đó chính là nội dung của các câu tiếp theo.
       </details>

    3. Xem lời giải

       <details>

        ```
        sed -i 's/controller-manager-XXXX.conf/controller-manager.conf/' /etc/kubernetes/manifests/kube-controller-manager.yaml
        ```

        > 💡 **Giải thích:** Xem log bằng `kubectl logs kube-controller-manager-controlplane -n kube-system` sẽ thấy lỗi không tìm thấy file kubeconfig `/etc/kubernetes/controller-manager-XXXX.conf`. File đúng là `/etc/kubernetes/controller-manager.conf` (kiểm tra bằng `ls /etc/kubernetes/`).
       </details>

    4. Xem lời giải

       <details>

        ```
        sed -i 's/WRONG-PKI-DIRECTORY/pki/' /etc/kubernetes/manifests/kube-controller-manager.yaml
        ```

        > 💡 **Giải thích:** Lần này log báo không tải được file certificate (vd `/etc/kubernetes/pki/ca.crt: no such file or directory`) dù các tham số trong `command` trông có vẻ đúng. Nguyên nhân nằm ở phần `volumes`: `hostPath` của volume chứa certificate trỏ sai thư mục trên node (`/etc/kubernetes/WRONG-PKI-DIRECTORY`), nên bên trong container không có file nào. Bài học: khi tham số đúng mà vẫn không thấy file, hãy kiểm tra `volumeMounts` và `volumes` của manifest.
       </details>




       