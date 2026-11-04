# Kết nối tới Cluster

1. Khi cluster đã được triển khai, sẽ có một dấu tích màu xanh lá ở cột status. Bây giờ bạn có thể bấm vào tên cluster, nếu bạn chưa bấm.<br/><br/>![select](../images/04-select.png)
1. Ở phía trên của màn hình tiếp theo, bấm vào `CONNECT`<br/><br/>![connect](../images/04-connect.png)
1. Bấm vào `RUN IN CLOUDSHELL`. Khung CloudShell sẽ mở ở phía dưới màn hình. Bấm nút `CONTINUE`.<br/><br/>![cloudshell](../images/04-run-in-cloudshell.png)
1. Terminal sẽ mở ra và một số lệnh sẽ xuất hiện trong đó, bấm `ENTER` để chạy chúng. Bấm `AUTHORIZE` ở hộp thoại bật ra.<br/><br/>![terminal](../images/04-terminal.png)
1. Kiểm tra bằng `kubectl`

    ```text
    kubectl get pods -A
    ```

> 💡 **Bổ sung:** Lệnh được điền sẵn trong CloudShell là `gcloud container clusters get-credentials <tên-cluster> --region us-west1 --project <project-id>`, có tác dụng ghi thông tin kết nối của cluster vào `~/.kube/config`. Bạn sẽ thấy các Pod hệ thống trong namespace `kube-system` (như `kube-dns`, `kube-proxy`, các agent giám sát của GKE...), nhưng không thấy Pod của kube-apiserver, etcd hay kube-scheduler vì control plane do Google quản lý. Trên tài khoản GCP thật, nhớ xóa cluster sau khi dùng (ví dụ `gcloud container clusters delete <tên-cluster> --region us-west1`) để tránh phát sinh chi phí.


