# DNS in Kubernetes

  - Xem [Bài giảng](https://kodekloud.com/topic/dns-in-kubernetes/)

Trong phần này, chúng ta sẽ tìm hiểu về **DNS trong Kubernetes cluster**.

> 💡 **Bổ sung:** Kubernetes triển khai một DNS server nội bộ (hiện nay mặc định là CoreDNS, thay cho kube-dns từ khoảng v1.11–v1.13; Service vẫn giữ tên `kube-dns` trong namespace `kube-system` để tương thích). Mặc định, các bản ghi DNS được tạo cho **Service**; bản ghi cho **Pod** dạng IP-có-gạch-ngang chỉ có khi CoreDNS bật tùy chọn `pods` trong plugin `kubernetes` (kubeadm mặc định có `pods insecure`). Domain gốc của cluster mặc định là `cluster.local`.

## Bản ghi DNS của Pod

- Định dạng phân giải DNS như sau:

```
<POD-IP-ADDRESS>.<namespace-name>.pod.cluster.local
```
> Ví dụ
```
# Pod nằm trong namespace default

10-244-1-10.default.pod.cluster.local
```

```
# Tạo một namespace
$ kubectl create ns apps

# Tạo một Pod
$ kubectl run nginx --image=nginx --namespace apps

# Xem thông tin chi tiết của Pod trong namespace "apps"
$ kubectl get po -n apps -owide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          99s   10.244.1.3   node01   <none>           <none>

# Tra bản ghi DNS của Pod nginx từ namespace default
$ kubectl run -it test --image=busybox:1.28 --rm --restart=Never -- nslookup 10-244-1-3.apps.pod.cluster.local
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      10-244-1-3.apps.pod.cluster.local
Address 1: 10.244.1.3
pod "test" deleted

# Truy cập bằng lệnh curl
$ kubectl run -it nginx-test --image=nginx --rm --restart=Never -- curl -Is http://10-244-1-3.apps.pod.cluster.local
HTTP/1.1 200 OK
Server: nginx/1.19.2

```

## Bản ghi DNS của Service

- Định dạng phân giải DNS như sau:

```
<service-name>.<namespace-name>.svc.cluster.local
```
> Ví dụ
```
# Service nằm trong namespace default

web-service.default.svc.cluster.local
```
- Pod và Service nằm trong namespace `apps`

```
# Expose Pod nginx
$ kubectl expose pod nginx --name=nginx-service --port 80 --namespace apps
service/nginx-service exposed

# Xem nginx-service trong namespace "apps"
$ kubectl get svc -n apps
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nginx-service   ClusterIP   10.96.120.174   <none>        80/TCP    6s

# Tra bản ghi DNS của nginx-service từ namespace default
$ kubectl run -it test --image=busybox:1.28 --rm --restart=Never -- nslookup nginx-service.apps.svc.cluster.local
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      nginx-service.apps.svc.cluster.local
Address 1: 10.96.120.174 nginx-service.apps.svc.cluster.local
pod "test" deleted

# Truy cập bằng lệnh curl
$ kubectl run -it nginx-test --image=nginx --rm --restart=Never -- curl -Is http://nginx-service.apps.svc.cluster.local
HTTP/1.1 200 OK
Server: nginx/1.19.2

```

> 💡 **Bổ sung:**
> - **Các dạng tên ngắn:** Pod trong cùng namespace có thể gọi Service chỉ bằng `nginx-service`; từ namespace khác dùng `nginx-service.apps`, `nginx-service.apps.svc` hoặc FQDN `nginx-service.apps.svc.cluster.local`. Điều này hoạt động nhờ dòng `search` trong `/etc/resolv.conf` của Pod (ví dụ `search default.svc.cluster.local svc.cluster.local cluster.local`).
> - **Tên Pod theo IP:** Bản ghi `10-244-1-3.apps.pod.cluster.local` dùng IP với dấu `.` được thay bằng `-`, không dùng tên Pod.
> - **Headless Service** (`clusterIP: None`): bản ghi A của Service trả về trực tiếp IP của các Pod backend; Pod trong StatefulSet còn có bản ghi dạng `<pod-name>.<service-name>.<namespace>.svc.cluster.local`.
> - **Mẹo thi:** Dùng image `busybox:1.28` cho `nslookup` như ví dụ trên, vì một số phiên bản busybox mới hơn có `nslookup` hoạt động không ổn định trong Kubernetes. Có thể lưu kết quả ra file theo yêu cầu đề, ví dụ `kubectl run test --image=busybox:1.28 --rm -it --restart=Never -- nslookup nginx-service.apps > /root/nslookup.out`.



#### Tài liệu tham khảo

- https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/
- https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/