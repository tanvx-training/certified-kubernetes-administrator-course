# ETCD cho người mới bắt đầu
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/etcd-for-beginners/)

  Trong phần này, chúng ta sẽ giới thiệu nhanh về ETCD cho người mới bắt đầu.
  - ETCD là gì?
  - Key-Value Store là gì?
  - Làm sao để bắt đầu nhanh với ETCD?
  - Làm sao để vận hành ETCD?

## ETCD là gì?
   - ETCD là một kho lưu trữ key-value phân tán, tin cậy, đơn giản, an toàn và nhanh.

> 💡 **Bổ sung:** "Phân tán" nghĩa là etcd có thể chạy thành một cụm nhiều member, dùng thuật toán đồng thuận **Raft** để đảm bảo dữ liệu nhất quán giữa các member. Cụm etcd cần đa số (quorum) member còn sống mới ghi được dữ liệu, vì vậy thường chạy số lẻ member (3 hoặc 5). Với cụm 3 member, bạn chịu được 1 member bị hỏng.

## Key-Value Store là gì?
   - Theo truyền thống, cơ sở dữ liệu có dạng bảng — chắc bạn đã nghe về SQL hay cơ sở dữ liệu quan hệ. Chúng lưu dữ liệu theo hàng và cột.

     ![relational-dbs](../../images/relational-dbs.PNG)

   - Key-Value Store lưu thông tin dưới dạng cặp Key (khóa) và Value (giá trị).

     ![key-value](../../images/key-value.PNG)

## Cài đặt ETCD
   - Cài đặt và bắt đầu dùng **`ETCD`** rất dễ.
     - Tải file binary phù hợp với hệ điều hành của bạn từ trang releases trên github (https://github.com/etcd-io/etcd/releases)

       Ví dụ: Để tải ETCD v3.5.6, chạy lệnh curl dưới đây

       ```
       $ curl -LO https://github.com/etcd-io/etcd/releases/download/v3.5.6/etcd-v3.5.6-linux-amd64.tar.gz
       ```
     - Giải nén.
       ```
       $ tar xvzf etcd-v3.5.6-linux-amd64.tar.gz
       ```
     - Chạy service ETCD
       ```
       $ ./etcd
       ```
     - Khi khởi động, mặc định **`ETCD`** sẽ lắng nghe ở port **`2379`**
      - Client mặc định đi kèm **`ETCD`** là [**`etcdctl`**](https://github.com/etcd-io/etcd/tree/main/etcdctl). Bạn có thể dùng nó để lưu và lấy các cặp key-value.
        ```
        Syntax: To Store a Key-Value pair
        $ ./etcdctl put key1 value1
        ```
        ```
        Syntax: To retrieve the stored data
        $ ./etcdctl get key1
        ```
        ```
        Syntax: To view more commands. Run etcdctl without any arguments
        $ ./etcdctl
        ```

        ![etcdctl](../../images/etcdctl.PNG)

> 💡 **Bổ sung:**
> - Port `2379` dùng cho client (kube-apiserver, `etcdctl`) kết nối tới; port `2380` dùng cho giao tiếp giữa các member (peer) trong cụm etcd.
> - **Phiên bản API của etcdctl:** etcdctl có hai phiên bản API là v2 và v3 với các lệnh khác nhau (v2 dùng `set`/`get`, v3 dùng `put`/`get`). Ở các bản etcd cũ (trước v3.4), etcdctl mặc định dùng API v2 nên phải đặt biến môi trường `ETCDCTL_API=3`. Từ etcd v3.4 trở đi, API v3 là mặc định — bạn vẫn thường thấy `ETCDCTL_API=3` trong tài liệu và lời giải, đặt thêm cũng không sao. Kiểm tra phiên bản bằng `etcdctl version`.
> - Từ etcd v3.5, các thao tác offline như `snapshot restore` được khuyến nghị thực hiện bằng công cụ `etcdutl` (lệnh `etcdctl snapshot restore` đã bị deprecated nhưng vẫn dùng được). Nội dung backup/restore etcd sẽ được nói chi tiết ở phần Cluster Maintenance.

Tài liệu tham khảo K8s:
- https://kubernetes.io/docs/concepts/overview/components/
- https://etcd.io/docs/
- https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/

