# Control Plane Failure (Lỗi Control Plane)

  - Xem [Bài giảng](https://kodekloud.com/topic/control-plane-failure/)
  
  - Trong bài giảng này, chúng ta sẽ tìm hiểu cách troubleshoot các thành phần của Control Plane.

  - Kiểm tra trạng thái các node xem có khỏe (healthy) không

    ```
    kubectl get nodes
    ```

  - Kiểm tra trạng thái các Pod xem có đang chạy không

    ```
    kubectl get pods
    ```

  - Kiểm tra trạng thái tất cả các Pod của các thành phần Control Plane (nếu cluster được triển khai bằng công cụ kubeadm) và đảm bảo chúng ở trạng thái **`Running`**

    ```
    kubectl get pods -n kube-system
    ```

    ![node](../../images/node.PNG)

    > 💡 **Bổ sung:** Với cluster kubeadm, kube-apiserver, kube-controller-manager, kube-scheduler và etcd chạy dưới dạng **static Pod**. kubelet đọc manifest của chúng trong thư mục `/etc/kubernetes/manifests/` (đường dẫn này được khai báo bởi `staticPodPath` trong `/var/lib/kubelet/config.yaml`). Muốn sửa một thành phần Control Plane, hãy sửa file manifest tương ứng — kubelet sẽ tự phát hiện thay đổi và tạo lại Pod (có thể mất 1–2 phút). Không dùng `kubectl edit` cho static Pod vì thay đổi sẽ không được lưu.

  - Nếu các thành phần Control Plane được triển khai dưới dạng service (cài đặt trực tiếp trên hệ điều hành) thì kiểm tra trạng thái của tất cả các thành phần

    ![cp](../../images/cp.PNG)

  - Kiểm tra trạng thái của **`kube-apiserver`** 

    ```
    service kube-apiserver status
    ```

  - Kiểm tra trạng thái của **`kube-controller-manager`** 

    ```
    service kube-controller-manager status
    ```

  - Kiểm tra trạng thái của **`kube-scheduler`** 

    ```
    service kube-scheduler status
    ```

    ![cp1](../../images/cp1.PNG)

  - Kiểm tra trạng thái của **`kubelet`** 

    ```
    service kubelet status
    ```

  - Kiểm tra trạng thái của **`kube-proxy`** trên các worker node.

    ```
    service kube-proxy status
    ```

    > 💡 **Bổ sung:** Trên các hệ điều hành dùng systemd hiện nay, lệnh tương đương và phổ biến hơn là `systemctl status <tên-service>` (vd `systemctl status kubelet`). Lưu ý: trong cluster kubeadm, chỉ **kubelet** (và container runtime như `containerd`) chạy dưới dạng systemd service; kube-proxy chạy dưới dạng DaemonSet trong namespace `kube-system`, nên kiểm tra bằng `kubectl get pods -n kube-system -l k8s-app=kube-proxy`.

  - Xem log của các thành phần Control Plane được triển khai dưới dạng Pod:

    ```
    kubectl logs kube-apiserver-master -n kube-system
    ```

    ![logs](../../images/logs.PNG)

  - Xem log của các thành phần Control Plane được triển khai dưới dạng SystemD Service

    ```
    sudo journalctl -u kube-apiserver
    ```

  > 💡 **Bổ sung — khi kube-apiserver bị hỏng thì `kubectl` không dùng được:** Lúc này mọi lệnh `kubectl` đều báo lỗi kiểu `The connection to the server ... was refused`. Hãy làm việc trực tiếp trên node controlplane với container runtime thông qua `crictl` (từ v1.24 dockershim đã bị gỡ, nên không dùng `docker ps` được nữa):
  > - `crictl ps -a | grep kube-apiserver` — liệt kê cả container đã thoát (Exited) để xem nó có đang restart liên tục không.
  > - `crictl logs <container-id>` — xem log của container (kể cả container đã chết) để tìm nguyên nhân, vd sai đường dẫn certificate, sai địa chỉ etcd, sai tham số.
  > - Log của Pod cũng được lưu trên đĩa tại `/var/log/pods/` và `/var/log/containers/`.
  > - Nếu container thậm chí không được tạo ra (vd YAML của manifest bị lỗi cú pháp), hãy xem log của kubelet: `journalctl -u kubelet -f` (hoặc `journalctl -u kubelet | grep -i apiserver`).
  >
  > Các lỗi hay gặp trong lab/đề thi: sai tên lệnh/binary trong `command`, sai đường dẫn file certificate/kubeconfig, sai `hostPath` của volume (ví dụ thư mục `pki`), sai port hoặc địa chỉ `--etcd-servers`.
