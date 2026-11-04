# ETCD trong môi trường HA

 - Xem [Bài giảng](https://kodekloud.com/topic/etcd-in-ha/)

> 💡 **Bổ sung:** Những điểm chính về etcd trong môi trường HA:
>
> - etcd là kho key-value phân tán, sử dụng thuật toán đồng thuận **Raft**. Trong cluster etcd có một member là **leader**, các member còn lại là follower. Mọi request ghi đều được chuyển tới leader. Leader chỉ coi một lần ghi là hoàn tất khi nó đã được sao chép tới **đa số (quorum)** member.
> - **Quorum = floor(N/2) + 1**, trong đó N là số member. Số member có thể hỏng mà cluster vẫn hoạt động (fault tolerance) là N - quorum:
>
>   | Số member | Quorum | Fault tolerance |
>   |---|---|---|
>   | 1 | 1 | 0 |
>   | 2 | 2 | 0 |
>   | 3 | 2 | 1 |
>   | 4 | 3 | 1 |
>   | 5 | 3 | 2 |
>   | 6 | 4 | 2 |
>   | 7 | 4 | 3 |
>
> - Vì vậy, khuyến nghị dùng **số lẻ** member, tối thiểu là 3 cho HA. Số chẵn không tăng fault tolerance và còn làm tăng nguy cơ mất quorum khi network bị phân mảnh (network partition). 5 member thường là mức hợp lý cho production. Số member quá lớn sẽ làm chậm việc ghi.
> - Mỗi member được cấu hình danh sách các peer qua flag `--initial-cluster` (cổng peer 2380, cổng client 2379). Có thể kiểm tra member bằng lệnh:
>
>   `ETCDCTL_API=3 etcdctl member list --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key`
>
>   (Từ etcd v3.4, `ETCDCTL_API=3` đã là mặc định. Đường dẫn certificate ở trên là đường dẫn mặc định trong cluster do kubeadm tạo.)
> - Xem thêm hai mô hình topology stacked etcd và external etcd tại bài [Configure High Availability](03-Configure-High-Availability.md).
