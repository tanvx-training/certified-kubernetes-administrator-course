# Service Networking

  - Xem [Bài giảng](https://kodekloud.com/topic/service-networking/)

Trong phần này, chúng ta sẽ tìm hiểu về **Service Networking** (mạng của Service).

> 💡 **Bổ sung:** Service là một khái niệm ảo trên toàn cluster: không có process hay interface mạng nào thực sự "lắng nghe" trên IP của Service. Khi Service được tạo, kube-apiserver cấp cho nó một IP từ dải `--service-cluster-ip-range`; kube-proxy chạy trên **mọi node** (DaemonSet) theo dõi Service và EndpointSlice, rồi tạo các rule (iptables, IPVS hoặc nftables) để chuyển tiếp traffic đi tới `ClusterIP:port` sang IP:port của một Pod backend.

## Các loại Service

- ClusterIP 


```
clusterIP.yaml

apiVersion: v1
kind: Service
metadata:
  name: local-cluster
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx
```

- NodePort

```
nodeportIP.yaml

apiVersion: v1
kind: Service
metadata:
  name: nodeport-wide
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx
```

## Tạo Service

```
$ kubectl create -f clusterIP.yaml
service/local-cluster created

$ kubectl create -f nodeportIP.yaml
service/nodeport-wide created
```

## Xem thông tin chi tiết của Pod

```
$ kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          1m   10.244.1.3   node01   <none>           <no
```

## Xem danh sách Service

```
$ kubectl get service
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP        5m22s
local-cluster   ClusterIP   10.101.67.139   <none>        80/TCP         3m
nodeport-wide   NodePort    10.102.29.204   <none>        80:30016/TCP   2m
```

## Kiểm tra dải Cluster IP của Service

```
$ ps -aux | grep kube-apiserver
--secure-port=6443 --service-account-key-file=/etc/kubernetes/pki/sa.pub --
service-cluster-ip-range=10.96.0.0/12

```

> 💡 **Bổ sung:** Với cluster dựng bằng kubeadm, cách nhanh hơn là đọc static Pod manifest: `grep service-cluster-ip-range /etc/kubernetes/manifests/kube-apiserver.yaml`. Dải IP Service không được trùng với dải IP Pod (`--cluster-cidr` / `--pod-network-cidr`) và mạng của node.

## Kiểm tra các rule do kube-proxy tạo trong iptables

```
$ iptables -L -t nat | grep local-cluster
KUBE-MARK-MASQ  all  --  10.244.1.3           anywhere             /* default/local-cluster: */
DNAT       tcp  --  anywhere             anywhere             /* default/local-cluster: */ tcp to:10.244.1.3:80
KUBE-MARK-MASQ  tcp  -- !10.244.0.0/16        10.101.67.139        /* default/local-cluster: cluster IP */ tcp dpt:http
KUBE-SVC-SDGXHD6P3SINP7QJ  tcp  --  anywhere             10.101.67.139        /* default/local-cluster: cluster IP */ tcp dpt:http
KUBE-SEP-GEKJR4UBUI5ONAYW  all  --  anywhere             anywhere             /* default/local-cluster: */
```

> 💡 **Bổ sung:** Luồng xử lý trong iptables: chain `KUBE-SERVICES` khớp ClusterIP:port → nhảy tới chain `KUBE-SVC-...` của Service → chọn ngẫu nhiên (theo xác suất) một chain `KUBE-SEP-...` (Service EndPoint) → `DNAT` tới IP:port của Pod. Với NodePort, có thêm chain `KUBE-NODEPORTS` khớp theo port trên node (mặc định trong dải `30000-32767`). Có thể xem chi tiết bằng `iptables -t nat -L KUBE-SERVICES -n`.

## Kiểm tra log của kube-proxy

- Vị trí file này có thể khác nhau tùy theo cách cài đặt.

```
$ cat /var/log/kube-proxy.log

```

> 💡 **Bổ sung:** Trong cluster kubeadm, kube-proxy chạy dưới dạng DaemonSet nên log được xem bằng `kubectl logs -n kube-system <kube-proxy-pod>` (thường không có file `/var/log/kube-proxy.log`). Chế độ proxy (`mode`: `iptables`, `ipvs`, hoặc `nftables` - beta từ v1.31) được cấu hình trong ConfigMap `kube-proxy` ở namespace `kube-system`; nếu để trống thì mặc định trên Linux là `iptables`. Dòng log như `Using iptables Proxier` cho biết chế độ đang dùng.


#### Tài liệu tham khảo

- https://kubernetes.io/docs/concepts/services-networking/service/