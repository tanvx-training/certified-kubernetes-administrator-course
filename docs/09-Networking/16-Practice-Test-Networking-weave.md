# Practice Test Networking Weave (Bài thực hành mạng với Weave)

  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-networking-weave/)

#### Lời giải

  1. <details>
      <summary>Cluster này có bao nhiêu node?</summary>

      ```bash
      kubectl get nodes
      ```

      > 2

     </details>

  2. <details>
      <summary>Cluster này đang dùng giải pháp mạng (Networking Solution) nào?</summary>

      Có hai cách để kiểm tra:

      1.
         ```bash
         kubectl get pods -n kube-system
         ```

      1.
         ```bash
         ls -l /opt/cni/bin
         ```

      Ở cả hai cách, bạn đều thấy dấu hiệu của

      > weave

      > 💡 **Bổ sung:** Cách đáng tin cậy nhất là xem file cấu hình CNI trên node: `ls /etc/cni/net.d/` (ví dụ `10-weave.conflist`). Thư mục `/opt/cni/bin` chứa binary của mọi plugin CNI đã cài (bridge, host-local, loopback...), nên có thể có nhiều plugin hơn plugin thực sự được dùng.

     </details>

  3. <details>
      <summary>Có bao nhiêu weave agent/peer được triển khai trong cluster này?</summary>

      ```bash
      kubectl get pods -n kube-system
      ```

      > 2

     </details>

  4. <details>
      <summary>Các weave peer nằm trên những node nào?</summary>

      ```bash
      kubectl get pods -n kube-system -o wide
      ```

      > Mỗi node một peer (One on every node)

      > 💡 **Bổ sung:** Weave được triển khai dưới dạng DaemonSet (`kubectl get ds -n kube-system`), nên mỗi node (kể cả control plane, nhờ có toleration phù hợp) chạy đúng một Pod `weave-net`.

     </details>

  5. <details>
      <summary>Xác định tên của bridge network/interface do weave tạo trên mỗi node.</summary>

      Trên bất kỳ host nào...

      ```bash
      ip addr list
      ```

      > weave

      Thực ra, network interface tên là `weave`, còn bridge được hiện thực bởi cặp `vethwe-datapath@vethwe-bridge` và `vethwe-bridge@vethwe-datapath`.

     </details>

  6. <details>
      <summary>Dải địa chỉ IP Pod do weave cấu hình là gì?</summary>

      Xem output của lệnh ở câu trước, tại interface `weave`. Để ý IP của nó bắt đầu bằng `10.`, vì vậy...

      > 10.X.X.X

      > 💡 **Bổ sung:** Bạn cũng có thể xem log của Pod weave để thấy dải IP cấp phát: `kubectl logs -n kube-system <weave-net-pod> -c weave | grep -i ipalloc-range`. Mặc định là `10.32.0.0/12`.

     </details>

  7. <details>
      <summary>Default gateway được cấu hình trên các Pod được schedule lên node01 là gì?</summary>

      Chúng ta có thể suy ra điều này từ câu trả lời của câu trước. Vì đã biết dải IP của weave, gateway phải nằm trong cùng mạng đó. Tuy nhiên, có thể kiểm chứng bằng cách chạy một Pod có sẵn công cụ `ip`.

      Hãy ghi nhớ [container image](https://github.com/wbitt/Network-MultiTool) này. Nó cực kỳ hữu ích khi debug các vấn đề mạng trong cluster!

      ```bash
      kubectl run testpod --image=wbitt/network-multitool
      ```

      Đợi Pod chuyển sang trạng thái Running.

      ```bash
      kubectl exec -it testpod -- ip route
      ```

      Để ý dòng đầu tiên của output. Đó chính là đáp án.

      > 💡 **Bổ sung:** Pod `testpod` phải chạy trên node01 thì kết quả mới đúng với câu hỏi. Kiểm tra bằng `kubectl get pod testpod -o wide`; nếu nó chạy trên node khác, có thể ép lên node01 bằng `--overrides='{"spec":{"nodeName":"node01"}}'` khi `kubectl run`. Gateway thường là IP của interface `weave` trên node01.
     </details>
