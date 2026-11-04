# Kiểm thử cluster

Ở bước này, chúng ta sẽ kiểm thử cluster bằng cách tạo một workload kèm một NodePort Service và lấy kết quả bằng `curl`

Thực hiện các bước sau trên `controlplane`

[//]: # (host:controlplane)

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

    Nếu bạn gặp lỗi này:
    ```
    curl: (7) Failed to connect to node01 port 30659 after 0 ms: Connection refused
    ```
    Hãy đợi vài giây rồi thử lại. Cả hai lệnh đều phải trả về trang chào mừng của nginx dưới dạng HTML.

    > 💡 **Bổ sung:** NodePort Service mở cùng một port (mặc định trong dải 30000-32767) trên **mọi** node, nên dù Pod nginx chỉ chạy trên một node, bạn vẫn gọi được qua cả `node01` và `node02` — kube-proxy sẽ chuyển tiếp traffic tới Pod. Nếu chỉ một node trả lời được, hãy kiểm tra CNI (Pod của calico trên node kia) và cấu hình `ip_forward`/`br_netfilter` ở bước chuẩn bị node.

Chúc mừng! Bạn đã có một cluster kubeadm hoạt động.

## Xem Service bằng trình duyệt

### VirtualBox hoặc Apple Silicon

Nếu bạn cài cluster với mạng bridge (mặc định), bạn có thể xem các NodePort Service bằng trình duyệt.

Chạy lệnh sau trên `controlplane` để lấy địa chỉ cho trình duyệt, rồi copy kết quả vào trình duyệt:

```bash
echo "http://$(dig +short node01):$PORT_NUMBER"
```

### AWS

1. Xem output của lần chạy terraform và lấy địa chỉ IP của `node01` hoặc `node02`. Nếu chưa ghi lại, bạn có thể tìm các địa chỉ IP trong [EC2 console](https://us-east-1.console.aws.amazon.com/ec2/home?region=us-east-1#Instances:) ở cột `Public IPv4 ...` (cuộn sang phải nếu không thấy cột này).
1. Ghép URL từ địa chỉ IP (của bạn sẽ khác) và số NodePort in ra ở bước 1 phía trên, ví dụ

    ```
    http://54.167.161.210:32182
    ```
1. Dán URL vào trình duyệt

Bạn cũng có thể copy KUBECONFIG sang `student-node` để chạy được các lệnh `kubectl` từ đó:

1. Quay lại `student-node`.
1. Chạy lệnh sau:

    ```bash
    mkdir ~/.kube
    scp controlplane:~/.kube/config ~/.kube/config
    kubectl get pods -n kube-system
    ```


Trước đó: [Các worker node](./06-workers.md)
