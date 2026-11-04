## Tạo một Service để kiểm thử

Chạy các lệnh sau trên `student-node`

1.  Đảm bảo tất cả các Pod calico đang chạy. Chúng có thể mất một lúc để khởi tạo

    ```bash
    watch kubectl get pods -n calico-system
    ```

    Nhấn `CTRL-C` để thoát watch khi các Pod đã ổn định

1. Lúc này cả 5 node phải ở trạng thái ready:

    ```
    kubectl get nodes
    ```

1. Deploy và expose một Pod nginx

    ```bash
    kubectl create deployment nginx --image nginx:alpine
    kubectl expose deploy nginx --type=NodePort --port 80

    PORT_NUMBER=$(kubectl get service -l app=nginx -o jsonpath="{.items[0].spec.ports[0].nodePort}")
    echo -e "\n\nService exposed on NodePort $PORT_NUMBER"

    ```

[//]: # (command:kubectl wait deployment -n default nginx --for condition=Available=True --timeout=90s)

2.  Gọi tới Service mới

    ```bash
    curl http://node01:$PORT_NUMBER
    curl http://node02:$PORT_NUMBER
    ```

    Cả hai lệnh đều phải trả về trang chào mừng của nginx dưới dạng HTML.

Chúc mừng! Bạn đã có một cluster kubeadm HA hoạt động.

> 💡 **Bổ sung:** Để thử tính sẵn sàng cao, hãy SSH vào một control plane (vd `controlplane02`) và tắt nó (`sudo shutdown -h now`, hoặc stop instance trong EC2 console). Các lệnh `kubectl` từ `student-node` vẫn chạy bình thường vì HAProxy chuyển request sang hai control plane còn lại, và etcd vẫn đủ quorum (2/3).

## Xem Service bằng trình duyệt

1. Xem output của lần chạy terraform và lấy địa chỉ IP của `node01` hoặc `node02`. Nếu chưa ghi lại, bạn có thể tìm các địa chỉ IP trong [EC2 console](https://us-east-1.console.aws.amazon.com/ec2/home?region=us-east-1#Instances:) ở cột `Public IPv4 ...` (cuộn sang phải nếu không thấy cột này).
1. Ghép URL từ địa chỉ IP (của bạn sẽ khác) và số NodePort in ra ở bước 1 phía trên, ví dụ

    ```
    http://54.167.161.210:32182
    ```
1. Dán URL vào trình duyệt
