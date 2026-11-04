# Practice Test - Deploy Networking Solution (Bài thực hành triển khai giải pháp mạng)

  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-deploy-network-solution/)

#### Lời giải

  1. <details>
      <summary>Chúng tôi đã triển khai một ứng dụng tên là app trong namespace default. Trạng thái của Pod là gì?</summary>

      ```bash
      kubectl get pods
      ```

      Để ý rằng Pod bị kẹt ở trạng thái `ContainerCreating` và sẽ cứ giữ nguyên như vậy.

      > NotRunning

     </details>

  2. <details>
      <summary>Hãy kiểm tra xem vì sao Pod không chạy.</summary>

      ```bash
      kubectl describe pod app
      ```

      Câu trả lời nằm trong phần `Events`. Pod không thể được cấp phát địa chỉ IP, do đó...

      > No network configured

      > 💡 **Bổ sung:** Khi cluster chưa cài plugin CNI, kubelet không thể thiết lập mạng cho Pod (sandbox), nên Pod kẹt ở `ContainerCreating` với event kiểu `network plugin is not ready: cni config uninitialized` hoặc `failed to setup network for sandbox`. Đồng thời các node thường ở trạng thái `NotReady` (`kubectl get nodes`) và các Pod CoreDNS ở trạng thái `Pending`. Đây là triệu chứng kinh điển của "chưa cài CNI" khi troubleshooting.

     </details>

  3. <details>
      <summary>Triển khai giải pháp mạng weave-net cho cluster.</summary>

      Apply manifest nằm trong thư mục `/root/weave`.

      > 💡 **Bổ sung:** Ví dụ: `ls /root/weave`, sau đó `kubectl apply -f /root/weave/` và kiểm tra bằng `kubectl get pods -n kube-system -l name=weave-net`.
      >
      > Sau khi các Pod `weave-net` chạy (DaemonSet, mỗi node một Pod), Pod `app` sẽ được cấp IP và chuyển sang `Running`. Lưu ý: Weaveworks đã ngừng hoạt động (2024) và Weave Net không còn được phát triển tích cực; trong thực tế và trong đề thi CKA mới, bạn thường gặp các CNI như Calico, Flannel hoặc Cilium. Quy trình vẫn giống nhau: apply manifest của CNI (theo hướng dẫn trong tài liệu được phép tra cứu), rồi kiểm tra node `Ready` và Pod chạy được. Hãy chú ý đảm bảo dải IP Pod của CNI khớp với `--pod-network-cidr` đã dùng khi `kubeadm init` (nếu có).
     </details>
