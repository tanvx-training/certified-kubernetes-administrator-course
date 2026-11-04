# Practice Test - Backup and Restore Methods 2 (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-backup-and-restore-methods-2-2/)

Lời giải cho bài practice test - Backup and Restore Methods 2

Trong bài này, chúng ta thực hành với cả hai kiểu etcd cluster: _stacked_ và _external_.

> 💡 **Bổ sung:**
> - **Stacked etcd**: etcd chạy ngay trên các node control plane, dưới dạng static Pod (do kubeadm tạo), mỗi control plane node có một member etcd. Dễ thiết lập nhưng nếu mất node thì mất cả member etcd lẫn control plane.
> - **External etcd**: etcd chạy trên các máy riêng, thường là systemd service, tách biệt khỏi control plane. kube-apiserver kết nối tới qua flag `--etcd-servers`. An toàn hơn nhưng cần nhiều máy hơn và cấu hình phức tạp hơn.

1. Chỉ là thông tin
1.  <details>
    <summary>Khám phá student-node và các cluster mà nó có quyền truy cập.</summary>

    ```bash
    kubectl config get-contexts
    ```

    </details>
1.  <details>
    <summary>Có bao nhiêu cluster được định nghĩa trong kubeconfig trên student-node?</summary>

    ```bash
    kubectl config get-contexts
    ```

    > 2

    </details>
1.  <details>
    <summary>Có bao nhiêu node (cả controlplane và worker) thuộc cluster1?</summary>

    ```bash
    kubectl config use-context cluster1
    kubectl get nodes
    ```

    > 2

    </details>
1.  <details>
    <summary>Tên của node controlplane trong cluster2 là gì?</summary>

    ```bash
    kubectl config use-context cluster2
    kubectl get nodes
    ```

    > cluster2-controlplane

    </details>
1. Chỉ là thông tin
1.  <details>
    <summary>ETCD được cấu hình như thế nào cho cluster1?</summary>

    ```bash
    kubectl config use-context cluster1
    kubectl get pods -n kube-system
    ```

    > Từ output, ta thấy có một Pod etcd, do đó đáp án là `Stacked ETCD`

    </details>
1.  <details>
    <summary>ETCD được cấu hình như thế nào cho cluster2?</summary>

    ```bash
    kubectl config use-context cluster2
    kubectl get pods -n kube-system
    ```

    > Từ output, ta không thấy Pod etcd nào. Vì một cluster _đang hoạt động_ thì không thể không có etcd, nên đáp án phải là `External ETCD`

    </details>
1.  <details>
    <summary>Địa chỉ IP của datastore External ETCD mà cluster2 sử dụng là gì?</summary>

    Để trả lời, ta cần xem xét cấu hình của API server

    ```bash
    kubectl config use-context cluster2
    kubectl get pods -n kube-system kube-apiserver-cluster2-controlplane -o yaml | grep etcd
    ```

    > Trong output, tìm `--etcd-servers`. Địa chỉ IP trong dòng này chính là đáp án.

    </details>
1.  <details>
    <summary>Thư mục dữ liệu mặc định (data directory) của datastore ETCD trong cluster1 là gì?</summary>

    Để trả lời, ta cần xem manifest của etcd trên node control plane, và tìm hostPath của volume `etcd-data`.

    ```bash
    kubectl config use-context cluster1
    kubectl get pods -n kube-system etcd-cluster1-controlplane -o yaml
    ```

    Trong output, tìm mục `volumes`. Host path của volume có tên `etcd-data` chính là đáp án.

    > /var/lib/etcd

    </details>
1. Chỉ là thông tin
1.  <details>
    <summary>Thư mục dữ liệu mặc định của datastore ETCD trong cluster2 là gì?</summary>

    Để trả lời, ta cần xem system unit file của dịch vụ etcd. Hãy nhớ rằng với external etcd, nó chạy dưới dạng một service của hệ điều hành.

    ```bash
    ssh etcd-server
    ```

    ```bash
    # Kiểm tra tên của service
    systemctl list-unit-files | grep etcd

    # Dùng kết quả từ lệnh trên
    systemctl cat etcd.service
    ```

    Chú ý dòng comment trong output. Dòng này cho biết service unit file nằm ở đâu. Chúng ta sẽ cần sửa file này ở một câu hỏi sau!

    Trong output, tìm `--data-dir`

    > /var/lib/etcd-data

    Quay lại student node:

    ```bash
    exit
    ```

    </details>
1.  <details>
    <summary>Có bao nhiêu node khác thuộc cùng cluster ETCD với etcd-server?</summary>

    Câu hỏi này có phần gây tranh cãi. Lẽ ra nó không nên chứa từ `other` (khác). Đáp án yêu cầu là

    > 1

    > 💡 **Bổ sung:** Có thể kiểm tra số member bằng lệnh sau trên etcd-server (đường dẫn cert lấy từ `systemctl cat etcd.service`):
    > ```bash
    > ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
    >   --cacert=<ca-file> --cert=<cert-file> --key=<key-file> \
    >   member list
    > ```

    </details>
1.  <details>
    <summary>Backup etcd của cluster1 và lưu trên student-node tại đường dẫn /opt/cluster1.db</summary>

    Để làm việc này, ta cần thực hiện backup trên node control plane, sau đó copy file về student node.

    ```bash
    ssh cluster1-controlplane
    ```

    ```bash
    ETCDCTL_API=3 etcdctl snapshot save \
      --cacert /etc/kubernetes/pki/etcd/ca.crt \
      --cert /etc/kubernetes/pki/etcd/server.crt \
      --key /etc/kubernetes/pki/etcd/server.key \
      cluster1.db

    # Quay lại student node
    exit
    ```

    ```bash
    scp cluster1-controlplane:~/cluster1.db /opt/
    ```

    </details>
1.  <details>
    <summary>Một bản backup ETCD của cluster2 được lưu tại /opt/cluster2.db. Dùng file snapshot này để restore cluster2 vào đường dẫn mới <b>/var/lib/etcd-data-new</b>.</summary>

    Như đã biết, `cluster2` đang dùng etcd _external_. Điều này có nghĩa là
    * `etcd` không nhất thiết phải nằm trên node control plane của cluster. Trong trường hợp này, nó không nằm ở đó.
    * `etcd` chạy như một service của hệ điều hành chứ không phải Pod, do đó không có file manifest nào để sửa. Thay vào đó, thay đổi được thực hiện trên service unit file.</br></br>

    Câu hỏi này gồm nhiều phần. Hãy đi qua lần lượt từng phần.

    1.  <details>
        <summary>Chuyển file backup sang node etcd-server</summary>

        ```bash
        scp /opt/cluster2.db etcd-server:~/
        ```
        </details>
    1.  <details>
        <summary>Đăng nhập vào node etcd-server</summary>

        ```bash
        ssh etcd-server
        ```

        </details>
    1.  <details>
        <summary>Kiểm tra quyền sở hữu (ownership) của thư mục etcd-data hiện tại</summary>

        Ta cần đảm bảo dữ liệu được restore có quyền sở hữu đúng. Vị trí của thư mục dữ liệu đã được xác định ở Q12

        ```bash
        ls -ld /var/lib/etcd-data/
        ```

        > Chú ý rằng owner và group đều là `etcd`
        </details>
    1.  <details>
        <summary>Thực hiện restore</summary>

        ```bash
        ETCDCTL_API=3 etcdctl snapshot restore \
            --data-dir /var/lib/etcd-data-new \
            cluster2.db
        ```

        > 💡 Tương đương với etcd v3.5+: `etcdutl snapshot restore cluster2.db --data-dir /var/lib/etcd-data-new`.

        </details>
    1.  <details>
        <summary>Đặt quyền sở hữu cho thư mục đã restore</summary>

        ```bash
        chown -R etcd:etcd /var/lib/etcd-data-new
        ```

        > 💡 **Giải thích:** Service etcd chạy dưới user `etcd`. Lệnh restore chạy bằng root nên thư mục mới thuộc root; nếu không `chown`, etcd sẽ không đọc/ghi được dữ liệu và service sẽ không khởi động được.

        </details>
    1.  <details>
        <summary>Cấu hình lại và restart etcd</summary>

        Ta cần biết vị trí của service unit file, vị trí này cũng đã được xác định ở Q12

        ```bash
        vi /etc/systemd/system/etcd.service
        ```

        Sửa tham số `--data-dir` thành thư mục vừa restore, rồi lưu lại.

        Cuối cùng, reload và restart service `etcd`. Mỗi khi sửa service unit file, cần chạy `daemon-reload` để `systemd` nạp lại cấu hình vào bộ nhớ.

        ```bash
        systemctl daemon-reload
        systemctl restart etcd.service
        ```

        Quay lại student node:

        ```bash
        exit
        ```

        > 💡 **Bổ sung:** Kiểm tra trạng thái bằng `systemctl status etcd` hoặc `journalctl -u etcd`. Nên cân nhắc restart các thành phần control plane (vd xóa Pod `kube-controller-manager` và `kube-scheduler` trong namespace `kube-system` để chúng được tạo lại, và restart kubelet trên cluster2-controlplane) để đảm bảo chúng không dùng dữ liệu đã cache.

        </details>
    1.  <details>
        <summary>Kiểm tra kết quả restore</summary>

        ```bash
        kubectl config use-context cluster2
        kubectl get all -n critical
        ```

        </details>
    </details>
