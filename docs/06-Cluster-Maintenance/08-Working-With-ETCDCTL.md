# Làm việc với ETCDCTL
  - Xem [Bài hướng dẫn](https://kodekloud.com/topic/working-with-etcdctl/)

> 💡 **Bổ sung:** Những điểm cần nhớ khi làm việc với `etcdctl`:
> - `etcdctl` có hai phiên bản API: v2 và v3. Từ etcd v3.4 trở đi, API v3 là mặc định nên biến môi trường `ETCDCTL_API=3` không còn bắt buộc; tuy nhiên đặt `export ETCDCTL_API=3` vẫn vô hại và an toàn khi không chắc phiên bản. Với API v2, các lệnh như `snapshot save` không tồn tại (v2 dùng `backup`).
> - Để làm việc với etcd của cluster kubeadm (có bật TLS), mọi lệnh đều cần chỉ định chứng chỉ và endpoint:
>   ```
>   ETCDCTL_API=3 etcdctl \
>     --endpoints=https://127.0.0.1:2379 \
>     --cacert=/etc/kubernetes/pki/etcd/ca.crt \
>     --cert=/etc/kubernetes/pki/etcd/server.crt \
>     --key=/etc/kubernetes/pki/etcd/server.key \
>     member list
>   ```
>   Đường dẫn chứng chỉ chính xác có thể lấy từ manifest `/etc/kubernetes/manifests/etcd.yaml` (các flag `--trusted-ca-file`, `--cert-file`, `--key-file`, `--listen-client-urls`).
> - Từ etcd v3.5, lệnh `etcdctl snapshot status` và `etcdctl snapshot restore` đã bị đánh dấu deprecated, thay bằng công cụ `etcdutl` (thao tác trực tiếp trên file, không cần kết nối tới etcd nên không cần chứng chỉ):
>   ```
>   etcdutl snapshot status /opt/snapshot.db -w table
>   etcdutl snapshot restore /opt/snapshot.db --data-dir /var/lib/etcd-from-backup
>   ```
>   `etcdctl snapshot save` vẫn dùng `etcdctl` vì cần kết nối tới etcd đang chạy. Trong kỳ thi, dùng công cụ nào có sẵn trên máy đều được.
