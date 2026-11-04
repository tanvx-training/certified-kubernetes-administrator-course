# Practice Test - CNI Weave (Bài thực hành)

  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-cni-weave/)

#### Lời giải

  1. <details>
      <summary>Kiểm tra service kubelet và xác định giá trị container runtime được thiết lập cho Kubernetes.</summary>

      Xem unit file của kubelet

      ```bash
      systemctl cat kubelet
      ```

      Chú ý dòng sau trong output

      ```
      EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
      ```

      Kiểm tra file này

      ```bash
      cat /var/lib/kubelet/kubeadm-flags.env
      ```

      Đáp án là giá trị của `--container-runtime`

      > REMOTE

      > 💡 **Bổ sung:** Flag `--container-runtime` (giá trị `remote`) đã bị deprecated từ v1.24 và bị loại bỏ ở v1.27, vì sau khi bỏ dockershim thì mọi runtime đều là "remote" (qua CRI). Trên các phiên bản mới, file này chỉ còn `--container-runtime-endpoint` (ví dụ `unix:///var/run/containerd/containerd.sock`) — nhìn vào đó để biết runtime là containerd hay CRI-O. Cũng có thể xem cột `CONTAINER-RUNTIME` trong `kubectl get nodes -o wide`.

      </details>

  2. <details>
      <summary>Đường dẫn nào được cấu hình chứa tất cả các file thực thi (binary) của các CNI plugin được hỗ trợ?</summary>

      Đây là vị trí tiêu chuẩn để cài đặt các CNI plugin

      | `/opt/cni/bin`

     </details>

  3. <details>
      <summary>Plugin nào dưới đây không có trong danh sách CNI plugin khả dụng trên host này?</summary>

      ```bash
      ls -l /opt/cni/bin
      ```

      Tìm trong các đáp án cho sẵn lựa chọn không xuất hiện trong output của lệnh trên

      > cisco

     </details>

  4. <details>
      <summary>CNI plugin nào được cấu hình để sử dụng trên Kubernetes cluster này?</summary>

      Trong các lựa chọn, cần nhận ra lựa chọn nào trong bốn cái không phải là tên của một nhà cung cấp mạng cho container. Trong ba cái còn lại, chỉ một cái có mặt trong `/opt/cni/bin`

      > flannel

      Lưu ý rằng `bridge` là một cơ chế để nối các mạng với nhau, không phải một _nhà cung cấp_ mạng (network provider).

      > 💡 **Bổ sung:** Cách chắc chắn nhất là xem file cấu hình trong `/etc/cni/net.d/` (ví dụ `ls /etc/cni/net.d/` sẽ thấy `10-flannel.conflist`) hoặc kiểm tra các Pod của CNI đang chạy: `kubectl get pods -A | grep -iE 'flannel|calico|cilium|weave'`.
     </details>

  5. <details>
      <summary>File thực thi nào sẽ được kubelet chạy sau khi một container và namespace liên quan của nó được tạo?</summary>

      Tiếp nối từ câu 4...

      > flannel

      Tất cả các file trong `/opt/cni/bin` đều là file thực thi làm các tác vụ liên quan đến cấu hình network namespace. Sau khi network namespace được cấu hình bằng các chương trình khác, `flannel` triển khai mạng.

      [Đây là một bài viết rất hay](https://tonylixu.medium.com/k8s-network-cni-introduction-b035d42ad68f) về công dụng của các chương trình trong `/opt/cni/bin`.

      > 💡 **Bổ sung:** Chính xác hơn, ở các phiên bản Kubernetes hiện tại, container runtime (containerd) mới là thành phần gọi CNI plugin, không phải kubelet. Plugin được gọi là giá trị `type` trong file cấu hình ở `/etc/cni/net.d/`. Với Flannel, plugin `flannel` sẽ ủy quyền (delegate) tiếp cho plugin `bridge` và IPAM `host-local` để tạo interface và cấp IP.
     </details>
