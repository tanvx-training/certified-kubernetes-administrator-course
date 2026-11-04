# CoreDNS in Kubernetes

  - Xem [Bài giảng](https://kodekloud.com/topic/coredns-in-kubernetes/)

Trong phần này, chúng ta sẽ tìm hiểu về **CoreDNS trong Kubernetes**.


## Xem các Pod

```
$ kubectl get pods -n kube-system
NAME                                      READY   STATUS    RESTARTS   AGE
coredns-66bff467f8-2vghh                  1/1     Running   0          53m
coredns-66bff467f8-t5nzm                  1/1     Running   0          53m
```

## Xem Deployment

```
$ kubectl get deployment -n kube-system
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
coredns                   2/2     2            2           53m
```

## Xem ConfigMap của CoreDNS

```
$ kubectl get configmap -n kube-system
NAME                                 DATA   AGE
coredns                              1      52m
```

## File cấu hình của CoreDNS (Corefile)

```
$ kubectl describe cm coredns -n kube-system

Corefile:
---
.:53 {
    errors
    health {       lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
    prometheus :9153
    forward . /etc/resolv.conf
    cache 30
    loop
    reload
}
```

> 💡 **Bổ sung:** Ý nghĩa các plugin chính trong Corefile:
> - `kubernetes cluster.local in-addr.arpa ip6.arpa`: plugin trả lời truy vấn cho domain của cluster (`cluster.local`) và truy vấn ngược (reverse lookup). `pods insecure` cho phép tạo bản ghi Pod dạng `10-244-1-3.<ns>.pod.cluster.local`.
> - `forward . /etc/resolv.conf`: mọi truy vấn không thuộc cluster (ví dụ `www.google.com`) được chuyển tiếp tới nameserver trong `/etc/resolv.conf` của node (vì Pod CoreDNS dùng `dnsPolicy: Default`).
> - `cache 30`: cache kết quả 30 giây; `reload`: tự nạp lại Corefile khi ConfigMap thay đổi (có độ trễ, có thể mất một lúc); `loop`: phát hiện vòng lặp chuyển tiếp DNS và dừng CoreDNS (Pod sẽ bị `CrashLoopBackOff`, một lỗi hay gặp khi troubleshooting).
> - Corefile được mount vào Pod CoreDNS từ ConfigMap `coredns` tại `/etc/coredns/Corefile`. Để sửa: `kubectl -n kube-system edit cm coredns`, rồi nếu cần áp dụng ngay thì `kubectl -n kube-system rollout restart deployment coredns`.

## Xem Service

```
$ kubectl get service -n kube-system
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   62m
```

## Xem cấu hình trong kubelet

```
$ cat /var/lib/kubelet/config.yaml | grep -A2  clusterDNS
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local

```

> 💡 **Bổ sung:** kubelet dùng `clusterDNS` và `clusterDomain` để sinh file `/etc/resolv.conf` cho mỗi Pod có `dnsPolicy: ClusterFirst` (giá trị mặc định). Khi troubleshooting DNS, hãy kiểm tra: Pod CoreDNS có `Running` không, Service `kube-dns` có endpoint không (`kubectl -n kube-system get endpoints kube-dns`), và IP trong `clusterDNS` có khớp với ClusterIP của Service `kube-dns` không.

## Xem tên miền đầy đủ (FQDN)

- Với lệnh `host`, chúng ta sẽ nhận được tên miền đầy đủ (fully qualified domain name - FQDN).

```
$ host web-service
web-service.default.svc.cluster.local has address 10.106.112.101

$ host web-service.default
web-service.default.svc.cluster.local has address 10.106.112.101

$ host web-service.default.svc
web-service.default.svc.cluster.local has address 10.106.112.101

$ host web-service.default.svc.cluster.local
web-service.default.svc.cluster.local has address 10.106.112.101
```

## Xem file `/etc/resolv.conf`

```
$ kubectl run -it --rm --restart=Never test-pod --image=busybox -- cat /etc/resolv.conf
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
pod "test-pod" deleted
```

> 💡 **Bổ sung:** `options ndots:5` nghĩa là: nếu tên cần phân giải có ít hơn 5 dấu chấm, resolver sẽ lần lượt thử ghép với các domain trong dòng `search` trước (ví dụ `web-service` → `web-service.default.svc.cluster.local`), sau đó mới thử tên gốc. Đó là lý do lệnh `host web-service` ở trên trả về được FQDN.

## Phân giải tên Pod

```
$ kubectl get pods -o wide
NAME      READY   STATUS    RESTARTS   AGE     IP           NODE     NOMINATED NODE   READINESS GATES
test-pod   1/1     Running   0          11m     10.244.1.3   node01   <none>           <none>
nginx      1/1     Running   0          10m     10.244.1.4   node01   <none>           <none>

$ kubectl exec -it test-pod -- nslookup 10-244-1-4.default.pod.cluster.local
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      10-244-1-4.default.pod.cluster.local
Address 1: 10.244.1.4 
```

## Phân giải tên Service

```
$ kubectl get service
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes    ClusterIP   10.96.0.1        <none>        443/TCP   85m
web-service   ClusterIP   10.106.112.101   <none>        80/TCP    9m

$ kubectl exec -it test-pod -- nslookup web-service.default.svc.cluster.local
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-service.default.svc.cluster.local
Address 1: 10.106.112.101 web-service.default.svc.cluster.local

```


#### Tài liệu tham khảo

- https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services
- https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#pods
