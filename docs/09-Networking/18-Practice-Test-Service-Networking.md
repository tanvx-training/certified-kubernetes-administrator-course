# Practice Test Service Networking (Bài thực hành mạng của Service)

  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-service-networking/)

#### Lời giải

1. <details>
   <summary>Các node trong cluster thuộc dải mạng nào?</summary>

   ```
   kubectl get nodes -o wide
   ```

   Xem cột INTERNAL-IP để suy ra:

   ```
   192.20.116.0/24
   ```

   > 💡 **Bổ sung:** IP cụ thể thay đổi theo từng lab. Cách chắc chắn hơn là xem prefix thực tế trên interface của node: `ip addr show eth0` (tìm dòng `inet <IP>/<prefix>`), rồi suy ra địa chỉ mạng.
   </details>

2. <details>
   <summary>Dải địa chỉ IP được cấu hình cho các Pod trong cluster này là gì?</summary>

   ```
   kubectl get pods -A -o wide
   ```

   Trong danh sách này, loại trừ các static Pod của control plane như `kube-apiserver` vì chúng chạy trên mạng của host (host network), không thuộc mạng Pod. Từ các Pod còn lại, ta suy ra:

   ```
   10.244.0.0/16
   ```

   > 💡 **Bổ sung:** Có thể xác định trực tiếp hơn qua cấu hình: log/cấu hình của CNI (ví dụ log Pod weave có `ipalloc-range`), flag `--cluster-cidr` trong `/etc/kubernetes/manifests/kube-controller-manager.yaml`, hoặc `kubectl get nodes -o jsonpath='{.items[*].spec.podCIDR}'` (nếu controller-manager có cấp podCIDR cho node).
   </details>

3. <details>
   <summary>Dải IP được cấu hình cho các Service trong cluster là gì?</summary>

   ```
   kubectl get service -A
   ```

   Xem cột CLUSTER-IP để suy ra:

   ```
   10.96.0.0/12
   ```

   > 💡 **Bổ sung:** Cách chính xác nhất là xem flag `--service-cluster-ip-range` của kube-apiserver: `grep service-cluster-ip-range /etc/kubernetes/manifests/kube-apiserver.yaml`. Chỉ nhìn các ClusterIP hiện có thì không suy ra được độ dài prefix một cách chắc chắn.
   </details>

4. <details>
   <summary>Có bao nhiêu Pod kube-proxy được triển khai trong cluster này?</summary>

   ```
   kubectl get pod -n kube-system | grep kube-proxy
   ```

   Đếm số kết quả.
   </details>

5. <details>
   <summary>kube-proxy được cấu hình dùng loại proxy nào?</summary>

   Từ output của câu trên, bạn có hai Pod kube-proxy, ví dụ:

   ```
   controlplane ~ kubectl get pod -n kube-system | grep kube-proxy
   kube-proxy-rtr8p                       1/1     Running   0             56m
   kube-proxy-t7w8f                       1/1     Running   0             56m
   ```

   Chọn một Pod bất kỳ và xem log của nó. Đáp án nằm ở đó.

   ```
   k logs -n kube-system kube-proxy-rtr8p
   ```

   > 💡 **Bổ sung:** Log sẽ có dòng như `Using iptables Proxier`, nên đáp án là `iptables`. Có thể kiểm tra cấu hình bằng `kubectl -n kube-system get cm kube-proxy -o yaml | grep mode`; nếu `mode: ""` thì kube-proxy dùng mặc định là iptables.
   </details>

6. <details>
   <summary>Kubernetes cluster này đảm bảo mỗi node đều chạy một Pod kube-proxy bằng cách nào?</summary>

   ```
   kubectl get all -n kube-system
   ```

   Từ output này, bạn có thể thấy `kube-proxy` là một `daemonset`.
   </details>

