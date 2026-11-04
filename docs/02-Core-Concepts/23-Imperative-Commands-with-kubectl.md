# Mẹo thi chứng chỉ - Lệnh Imperative với kubectl
  - Đi tới [trang Mẹo thi chứng chỉ](https://kodekloud.com/topic/certification-tips-imperative-commands-with-kubectl/)

> 💡 **Bổ sung:** Tóm tắt các lệnh imperative hay dùng trong kỳ thi (giúp tiết kiệm thời gian so với viết YAML từ đầu):
> - `--dry-run=client` chỉ kiểm tra lệnh, không tạo resource; kết hợp với `-o yaml` để sinh file YAML mẫu: `kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml`
> - Tạo Pod: `kubectl run nginx --image=nginx` (thêm `--labels=tier=db`, `--port=8080`, `-n <namespace>` khi cần)
> - Tạo Deployment: `kubectl create deployment nginx --image=nginx --replicas=4`
> - Scale Deployment: `kubectl scale deployment nginx --replicas=4`
> - Expose Pod thành Service ClusterIP: `kubectl expose pod redis --port=6379 --name=redis-service` (tự dùng label của Pod làm selector)
> - Tạo Service ClusterIP: `kubectl create service clusterip redis --tcp=6379:6379` (selector mặc định là `app=redis`)
> - Expose Pod thành Service NodePort: `kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service` (không chỉ định được `nodePort` - sinh YAML rồi sửa)
> - Tạo Service NodePort: `kubectl create service nodeport nginx --tcp=80:80 --node-port=30080` (selector mặc định là `app=nginx`)
> - Tạo Pod kèm Service cùng tên: `kubectl run httpd --image=httpd:alpine --port=80 --expose`
>
> Tham khảo: https://kubernetes.io/docs/reference/kubectl/conventions/ và https://kubernetes.io/docs/reference/kubectl/quick-reference/
