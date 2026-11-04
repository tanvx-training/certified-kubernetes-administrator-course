# Practice Test - Backup and Restore Methods (Bài thực hành)
Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-backup-and-restore-methods/)

Lời giải cho bài practice test - Backup and Restore Methods

1.  <details>
    <summary>Có bao nhiêu Deployment trong cluster?</summary>

    ```
    kubectl get deployments
    ```
    </details>

1.  <details>
    <summary>Phiên bản ETCD đang chạy trên cluster là gì?</summary>

    ```
    kubectl describe pod -n kube-system etcd-controlplane
    ```

    Tìm dòng `Image` (phần tag của image chính là phiên bản etcd)
    </details>

1.  <details>
    <summary>Từ node controlplane, có thể kết nối tới cluster ETCD qua địa chỉ nào?</summary>

    ```
    kubectl describe pod -n kube-system etcd-controlplane
    ```

    Trong mục `Command`, tìm `--listen-client-urls`
    </details>

1.  <details>
    <summary>File certificate của ETCD server nằm ở đâu?</summary>

    Với cluster kubeadm như cluster này, vị trí mặc định của các file certificate là `/etc/kubernetes/pki/etcd`

    Chọn đúng certificate (giá trị của flag `--cert-file` trong manifest etcd, thường là `/etc/kubernetes/pki/etcd/server.crt`)
    </details>

1.  <details>
    <summary>File CA Certificate của ETCD nằm ở đâu?</summary>

    Với cluster kubeadm như cluster này, vị trí mặc định của các file certificate là `/etc/kubernetes/pki/etcd`

    Chọn đúng certificate (giá trị của flag `--trusted-ca-file`, thường là `/etc/kubernetes/pki/etcd/ca.crt`)
    </details>

1.  <details>
    <summary>Tạo snapshot cơ sở dữ liệu ETCD bằng tính năng snapshot có sẵn.</br>Lưu file backup tại <b>/opt/snapshot-pre-boot.db</b></summary>

    ```
    ETCDCTL_API=3 etcdctl snapshot save \
      --cacert=/etc/kubernetes/pki/etcd/ca.crt \
      --cert=/etc/kubernetes/pki/etcd/server.crt \
      --key=/etc/kubernetes/pki/etcd/server.key \
      /opt/snapshot-pre-boot.db
    ```

    > 💡 **Bổ sung:** Không cần `--endpoints` vì etcd lắng nghe trên `127.0.0.1:2379` (giá trị mặc định). Có thể kiểm tra snapshot bằng `etcdutl snapshot status /opt/snapshot-pre-boot.db -w table` (hoặc `etcdctl snapshot status ...` trên bản etcd cũ).
    </details> 

1. Chỉ là thông tin.


1.  <details>
    <summary>Dậy đi! Có cuộc họp khẩn! Sau khi reboot, các master node đã online trở lại, nhưng không ứng dụng nào truy cập được. Hãy kiểm tra trạng thái các ứng dụng trên cluster. Có vấn đề gì?</summary>

    > All of the above (Tất cả các đáp án trên – Deployment, Service, Pod đều biến mất)
    </details>

1.  <details>
    <summary>May mắn là chúng ta đã có bản backup. Hãy restore lại trạng thái ban đầu của cluster bằng file backup.</summary>

    1. Restore bản backup vào một thư mục mới

        ```
        ETCDCTL_API=3 etcdctl snapshot restore \
          --data-dir /var/lib/etcd-from-backup \
          /opt/snapshot-pre-boot.db
        ```

        > 💡 Với etcd v3.5+, lệnh tương đương được khuyến nghị là `etcdutl snapshot restore /opt/snapshot-pre-boot.db --data-dir /var/lib/etcd-from-backup`. Lệnh restore làm việc trực tiếp trên file nên không cần cert. Thư mục đích không được tồn tại sẵn (hoặc phải rỗng).

    1. Sửa Pod `etcd` để dùng thư mục mới.

        Để làm việc này, cần sửa mục `volumes` và đổi `hostPath` thành thư mục mà ta vừa restore vào.

        ```
        vi /etc/kubernetes/manifests/etcd.yaml
        ```

        ```yaml
          volumes:
          - hostPath:
              path: /etc/kubernetes/pki/etcd
              type: DirectoryOrCreate
            name: etcd-certs
          - hostPath:
              path: /var/lib/etcd      # <- thay đổi giá trị này
              type: DirectoryOrCreate
            name: etcd-data
        ```

        Giá trị mới: `/var/lib/etcd-from-backup`

        Lưu lại và chờ tối đa khoảng một phút để Pod `etcd` được tạo lại.

        > 💡 **Giải thích:** Vì etcd là static Pod, kubelet theo dõi thư mục `/etc/kubernetes/manifests` và tự tạo lại Pod khi file manifest thay đổi. Chỉ cần đổi `hostPath` là đủ: thư mục trên host được mount vào container tại `/var/lib/etcd`, nên `--data-dir` bên trong container không cần sửa. Trong lúc etcd khởi động lại, `kubectl` có thể tạm thời không phản hồi; có thể theo dõi bằng `crictl ps | grep etcd`. Nếu Pod etcd không lên được, kiểm tra log bằng `crictl logs <container-id>` hoặc `journalctl -u kubelet`.

    1. Kiểm tra lại

        ```
        kubectl get deployments
        kubectl get services
        ```
    </details>

Xem thêm: https://github.com/kodekloudhub/community-faq/blob/main/docs/etcd-faq.md
