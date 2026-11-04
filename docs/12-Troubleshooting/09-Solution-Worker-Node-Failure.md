# Lời giải Worker Node Failure

  - Cùng xem lời giải cho [Practice Test](https://kodekloud.com/topic/practice-test-worker-node-failure/) về Worker Node Failure

   ### Lời giải

   1. <details>
      <summary>Sửa cluster đang bị hỏng</summary>

      * Sửa `node01`

      1. Kiểm tra các node

         ```bash
         kubectl get nodes
         ```

         Ta thấy `node01` có trạng thái `NotReady`. Điều này thường có nghĩa là đã mất liên lạc với kubelet của node đó.

      1. SSH vào node để điều tra

         ```bash
         ssh node01
         ```

      1. Kiểm tra trạng thái kubelet

         ```bash
         systemctl status kubelet
         ```

         Từ output có thể thấy kubelet không chạy, thực tế là nó đã thoát (exited). Vì vậy ta thử khởi động nó.

      1. Khởi động kubelet

         ```bash
         systemctl start kubelet
         ```

      1. Kiểm tra lại xem đã ổn chưa.

         ```bash
         systemctl status kubelet
         ```

         Giờ ta thấy trạng thái là `active (running)`, vậy là tốt.

         > 💡 **Bổ sung:** Nếu kubelet bị dừng và ở trạng thái `disabled`, nên chạy thêm `systemctl enable kubelet` để nó tự khởi động lại khi node reboot.

      1. Quay về controlplane

         ```bash
         exit
         ```

      1. Kiểm tra lại các node

         ```bash
         kubectl get nodes
         ```

         Đã ổn!

      </details>

   2. <details>
      <summary>Cluster lại bị hỏng. Hãy điều tra và sửa lỗi.</summary>

      * Sửa cluster

      1. Kiểm tra các node

         ```bash
         kubectl get nodes
         ```

         Ta thấy `node01` có trạng thái `NotReady`. Điều này thường có nghĩa là đã mất liên lạc với kubelet của node đó.

      1. SSH vào node để điều tra

         ```bash
         ssh node01
         ```

      1. Kiểm tra trạng thái kubelet

         ```bash
         systemctl status kubelet
         ```

         Từ output có thể thấy nó đang bị crashloop (`activating (auto-restart)`), vì vậy nhiều khả năng đây là lỗi cấu hình.

      1. Xem log của kubelet

         ```bash
         journalctl -u kubelet
         ```

         Có rất nhiều thông tin, nhưng lỗi mà ta quan tâm — nguyên nhân gốc của tất cả các lỗi khác — là lỗi sau

         ```
         "failed to construct kubelet dependencies: unable to load client CA file /etc/kubernetes/pki/WRONG-CA-FILE.crt: open /etc/kubernetes/pki/WRONG-CA-FILE.crt: no such file or directory"
         ```

         Nếu kubelet không tải được certificate thì nó không thể xác thực với API server. Đây là lỗi nghiêm trọng (fatal), nên kubelet thoát.

         > 💡 **Mẹo:** Output của `journalctl` rất dài. Nhấn `Shift+G` (trong pager `less`) để nhảy xuống cuối, hoặc dùng `journalctl -u kubelet --no-pager | tail -30`, hay lọc lỗi bằng `journalctl -u kubelet | grep -iE "error|failed" | tail`.

      1.  Kiểm tra thư mục certificate được chỉ ra

            ```bash
            ls -l /etc/kubernetes/pki
            ```

            Ta thấy thư mục này chứa `ca.crt`, ta giả định đó là certificate đúng, vì vậy cần tìm file cấu hình của kubelet và sửa lỗi ở đó.

      1. Tìm file cấu hình của kubelet

         kubelet là một service của hệ điều hành, nên file unit của service sẽ cho ta thông tin đó

         ```bash
         systemctl cat kubelet
         ```

         Để ý dòng này

         ```
         Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
         ```

         Đó chính là file YAML cấu hình.

      1. Sửa cấu hình

         ```bash
         vi /var/lib/kubelet/config.yaml
         ```

         ```yaml
         apiVersion: kubelet.config.k8s.io/v1beta1
         authentication:
         anonymous:
           enabled: false
         webhook:
           cacheTTL: 0s
           enabled: true
         x509:
           clientCAFile: /etc/kubernetes/pki/WRONG-CA-FILE.crt # <- Sửa chỗ này
         authorization:
         mode: Webhook
         ```

         Lưu ý rằng bạn có thể thực hiện cùng thao tác sửa này chỉ bằng một lệnh `sed`. Cách này nhanh hơn sửa bằng vi.

         ```bash
         sed -i 's/WRONG-CA-FILE.crt/ca.crt/g' /var/lib/kubelet/config.yaml
         ```

      1. Kiểm tra trạng thái

         Đợi vài giây, kubelet sẽ được tự động khởi động lại.

         ```bash
         systemctl status kubelet
         ```

         Giờ ta thấy trạng thái là `active (running)`, vậy là tốt. Nếu chưa được, có thể bạn đã sửa file cấu hình sai, nhiều khả năng là làm hỏng cú pháp YAML hoặc sửa tên file certificate chưa đúng. Quay lại bước `vii.` ở trên và sửa lại.

      1. Quay về controlplane

         ```bash
         exit
         ```

      1. Kiểm tra lại các node

         ```bash
         kubectl get nodes
         ```

         Đã ổn!
      </details>

   3. <details>
      <summary>Cluster lại bị hỏng. Hãy điều tra và sửa lỗi.</summary>

      * Sửa cluster

      1. Kiểm tra các node

         ```bash
         kubectl get nodes
         ```

         Ta thấy `node01` có trạng thái `NotReady`. Điều này thường có nghĩa là đã mất liên lạc với kubelet của node đó.

      1. SSH vào node để điều tra

         ```bash
         ssh node01
         ```

      1. Kiểm tra trạng thái kubelet

         ```bash
         systemctl status kubelet
         ```

         Ta thấy nó đang `active (running)`, nhưng API server vẫn cho rằng có vấn đề. Vì vậy một lần nữa ta phải xem log của kubelet.

      1. Xem log của kubelet

         ```bash
         journalctl -u kubelet
         ```

         Có rất nhiều thông tin, nhưng lỗi mà ta quan tâm — nguyên nhân gốc của tất cả các lỗi khác — là lỗi sau

         ```
          "Unable to register node with API server" err="Post \"https://controlplane:6553/api/v1/nodes\": dial tcp 192.10.46.12:6553: connect: connection refused" node="node01"
          ```

         Port thông thường của API server là bao nhiêu? Không phải `6553`! kubelet dùng một file kubeconfig để kết nối tới API server giống như kubectl, vì vậy ta cần tìm và sửa file đó.

      1. Tìm file kubeconfig của kubelet

         kubelet là một service của hệ điều hành, nên file unit của service sẽ cho ta thông tin đó

         ```bash
         systemctl cat kubelet
         ```

         Để ý dòng này

         ```
         Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
         ```

         Có hai file kubeconfig. File thứ nhất được dùng khi node mới được tạo và đang join vào cluster (TLS bootstrapping). File thứ hai được dùng khi hoạt động bình thường. Do đó file thứ hai mới là file ta quan tâm.

      1. Sửa kubeconfig

         Port phải là `6443`

         ```bash
         vi /etc/kubernetes/kubelet.conf
         ```

         ```yaml
         apiVersion: v1
         clusters:
         - cluster:
            certificate-authority-data: REDACTED
            server: https://controlplane:6553  # <- Sửa chỗ này
         name: default-cluster
         contexts:
         - context:
            cluster: default-cluster
            namespace: default
            user: default-auth
         name: default-context
         current-context: default-context
         kind: Config
         preferences: {}
         users:
         - name: default-auth
         user:
            client-certificate: /var/lib/kubelet/pki/kubelet-client-current.pem
            client-key: /var/lib/kubelet/pki/kubelet-client-current.pem
         ```

         Lưu ý rằng bạn có thể thực hiện cùng thao tác sửa này chỉ bằng một lệnh `sed`. Cách này nhanh hơn sửa bằng vi.

         ```bash
         sed -i 's/6553/6443/g' /etc/kubernetes/kubelet.conf
         ```

         > 💡 **Mẹo:** Nếu không nhớ port đúng, hãy so sánh với kubeconfig trên controlplane (`grep server ~/.kube/config` hoặc `/etc/kubernetes/admin.conf`), hoặc xem tham số `--secure-port` trong `/etc/kubernetes/manifests/kube-apiserver.yaml`.

      1. Khởi động lại kubelet

         Vì kubelet đang chạy (không bị crashloop), ta cần restart nó để nó đọc kubeconfig đã được cập nhật

         ```bash
         systemctl restart kubelet
         ```

      1. Kiểm tra trạng thái

         ```bash
         systemctl status kubelet
         ```

         Giờ ta thấy trạng thái là `active (running)`, vậy là tốt. Nếu chưa được, có thể bạn đã sửa kubeconfig sai, nhiều khả năng là làm hỏng cú pháp YAML. Quay lại bước `vi.` ở trên và sửa lại.

      1. Quay về controlplane

         ```bash
         exit
         ```

      1. Kiểm tra lại các node

         ```bash
         kubectl get nodes
         ```

         Đã ổn! Nếu chưa, có lẽ bạn đã đặt sai số port. Quay lại `node01` và làm lại từ bước `vi.` ở trên.

      </details>
