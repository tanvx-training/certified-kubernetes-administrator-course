# Join các worker node

1.  SSH tới `node01`
1.  Chuyển sang root

    ```bash
    sudo -i
    ```

1. Dán lệnh join dành cho các node *worker* mà `kubeadm init` đã in ra trên `controlplane01`

1. Quay về `student-node`

    ```
    exit
    exit
    ```

1. Lặp lại các bước 2, 3 và 4 trên `node02`

1. Giờ bạn đã quay lại `student-node`. Kiểm tra tất cả các node đã lên

    ```bash
    kubectl get nodes -o wide
    ```

    Lúc này sẽ có 3 control node và 2 worker.

    > 💡 **Bổ sung:** Các node control plane do kubeadm tạo có taint `node-role.kubernetes.io/control-plane:NoSchedule`, nên Pod ứng dụng thông thường sẽ chỉ được schedule lên `node01` và `node02`.

Tiếp theo: [Kiểm thử](./08-test.md)</br>
Trước đó: [Thiết lập Control Plane](./06-controlplane.md)
