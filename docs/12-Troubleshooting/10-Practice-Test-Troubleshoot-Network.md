# Lời giải Troubleshoot Network

Cùng xem lời giải cho [Practice Test](https://kodekloud.com/topic/practice-test-troubleshoot-network/) về Troubleshoot Network

Lưu ý rằng lab này mang tính tuần tự. Bạn phải giải xong hoàn toàn test 1 thì mới giải được test 2, nghĩa là không thể bỏ qua test 1 để chỉ làm test 2.

1. <details>
   <summary>Troubleshooting Test 1</summary>

   Đề bài yêu cầu đảm bảo tất cả các thành phần đều hoạt động, vì vậy trước tiên hãy kiểm tra cluster xem nó đang ở trạng thái nào.

   Có bao nhiêu node, và trạng thái của chúng?

   ```
   kubectl get nodes
   ```

   Có vẻ ổn...

   Tiếp theo, các Pod

   ```
   kubectl get pods -A
   ```

   Giờ ta thấy các Pod `webapp` và `mysql` bị kẹt ở trạng thái `ContainerCreating`. Cần describe các Pod và kiểm tra lỗi.

   Bạn sẽ thấy chúng báo lỗi `plugin type="weave-net" name="weave" failed (add): unable to allocate IP address`, vậy rõ ràng là có vấn đề về mạng và nó liên quan tới Weave.

   Khi chạy `get pods` ở trên, bạn có thấy dấu hiệu nào của các container hỗ trợ mạng, như `weave`, không?

   Không — vậy ta cần cài đặt giải pháp mạng (CNI plugin).

   Hãy cài đặt `Weave`

   ```
   kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
   ```

   Giờ đợi khoảng một phút để nó khởi tạo, rồi kiểm tra các Pod của ứng dụng

   ```
   kubectl get pods -n triton
   ```

   > 💡 **Bổ sung:** Lỗi ở giai đoạn `ContainerCreating` liên quan tới "sandbox"/"network plugin" gần như luôn là lỗi CNI. Những chỗ cần kiểm tra trên node: file cấu hình CNI trong `/etc/cni/net.d/` và binary plugin trong `/opt/cni/bin/`. Lưu ý Weave Net đã ngừng phát triển (công ty Weaveworks đóng cửa đầu năm 2024, repository đã được archive) nên các cluster mới thường dùng Calico, Cilium hoặc Flannel; tuy nhiên trong lab này vẫn dùng Weave. Khi đi thi, nếu được yêu cầu cài CNI, đề bài thường cung cấp sẵn link/manifest cần dùng.

   </details>


1. <details>
   <summary>Troubleshooting Test 2</summary>

   Một lần nữa, hãy kiểm tra cluster xem nó đang ở trạng thái nào.

   Có bao nhiêu node, và trạng thái của chúng?

   ```
   kubectl get nodes
   ```

   Có vẻ ổn...

   Tiếp theo, các Pod

   ```
   kubectl get pods -A
   ```

   Pod kube-proxy không chạy. Thực tế là nó bị crash-loop, nghĩa là nó cố khởi động rồi thất bại, lặp đi lặp lại. Hệ quả là các rule (iptables/IPVS) cần thiết để cho phép kết nối tới các Service chưa được tạo ra. Nơi đầu tiên cần xem khi chẩn đoán CrashLoopBackoff là log của Pod.

   1. Xem log của Pod kube-proxy

      ```
      kubectl -n kube-system logs <name_of_the_kube_proxy_pod>
      ```

      Ta thấy nó không tìm được file cấu hình.

      Giờ thử tìm file cấu hình, phòng trường hợp nó có tên khác

      ```
      ls -l /var/lib/kube-proxy
      ```

      Không tìm thấy thư mục này! (Vì thư mục đó chỉ tồn tại bên trong container, không phải trên node.)

   1. Kiểm tra pod template spec trong DaemonSet `kube-proxy`.

      ```
      kubectl get ds -n kube-system kube-proxy -o yaml | less
      ```

      Cuộn qua và kiểm tra phần volumes và volume mounts. Để ý rằng có một ConfigMap được mount tại đường dẫn `/var/lib/kube-proxy` bên trong Pod.

   1. Kiểm tra ConfigMap

      ```
      kubectl describe cm -n kube-system kube-proxy
      ```

      Ở đây ta thấy các file được ConfigMap mount vào là `config.conf` và `kubeconfig.conf`, nhưng _không có_ `configuration.conf`.

      Hai file này là

      * `config.conf` - Đây là cấu hình thực sự mà kube-proxy cần tải. File này tham chiếu tới `kubeconfig.conf`
      * `kubeconfig.conf` - Đây đơn giản là một file kubeconfig, giống như file bạn thấy trên terminal của lab tại `~/.kube/config`. Nó chứa thông tin xác thực và địa chỉ để kube-proxy giao tiếp với API server.

   1. Sửa tham số dòng lệnh của `kube-proxy`

      ```
      kubectl edit ds -n kube-system kube-proxy
      ```

      Đặt lại đúng tên file

      ```
      --config=/var/lib/kube-proxy/config.conf
      ```

      Cuối cùng, xác nhận rằng nó đang chạy.

      ```
      kubectl get pods -n kube-system
      ```

      > 💡 **Bổ sung:** Sau khi sửa DaemonSet, các Pod kube-proxy sẽ được tạo lại tự động (rolling update của DaemonSet). Nếu Pod vẫn chưa được thay, có thể ép tạo lại bằng `kubectl -n kube-system rollout restart ds kube-proxy` hoặc `kubectl -n kube-system delete pod -l k8s-app=kube-proxy`. Ngoài kube-proxy, cũng nên kiểm tra CoreDNS (`kubectl get pods -n kube-system -l k8s-app=kube-dns`) khi gặp lỗi kết nối giữa các Service.

   </details>
