# Khởi động các worker node

Ở bước này, chúng ta sẽ join các worker node vào cluster. Bạn sẽ cần lệnh `kubeadm join` từ bước trước.

Tham khảo: https://stackoverflow.com/questions/59629319/unable-to-upgrade-connection-pod-does-not-exist

## Join các worker

Nếu bạn chưa ghi lại lệnh join trên node controlplane sau khi chạy `kubeadm`, bạn có thể lấy lại bằng cách chạy lệnh sau trên `controlplane`

```bash
kubeadm token create --print-join-command
```

[//]: # (host:node01-node02)
[//]: # (comment:Run kubeadm join)

Trên từng node `node01` và `node02`, thực hiện như sau

1.  Chuyển sang user root (nếu chưa phải root)

    ```
    sudo -i
    ```

1.  Join node

    > Dán lệnh `kubeadm join` mà `kubeadm init` đã in ra trên control plane

    > 💡 **Bổ sung:** Lệnh join có dạng `kubeadm join <IP>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>`. `--token` dùng để node xác thực với API server khi bootstrap (TLS bootstrapping), còn `--discovery-token-ca-cert-hash` giúp node xác minh rằng nó đang nói chuyện với đúng control plane (chống giả mạo). Sau khi join, kubelet trên worker nhận được client certificate riêng và ghi vào `/etc/kubernetes/kubelet.conf`.

### Kiểm tra

Chạy lệnh sau trên `controlplane`. Sau vài giây, cả hai node sẽ ở trạng thái Ready

```
kubectl get nodes
```

[Link tài liệu Kubernetes](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/adding-linux-nodes/)


Tiếp theo: [Kiểm thử](./07-test.md)</br>
Trước đó: [Khởi động controlplane](./05-controlplane.md)
