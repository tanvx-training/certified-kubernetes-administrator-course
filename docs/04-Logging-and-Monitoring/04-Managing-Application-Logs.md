# Quản lý log của ứng dụng (Managing Application Logs)
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/managing-application-logs/)

Trong phần này, chúng ta sẽ tìm hiểu cách quản lý log của ứng dụng.

#### Hãy bắt đầu với logging trong Docker

![ld](../../images/ld.PNG)

Container `event-simulator` sinh ra các sự kiện ngẫu nhiên và ghi chúng ra standard output (stdout). Khi chạy container ở chế độ detached (`docker run -d`), log sẽ không hiện trên terminal; ta xem lại bằng lệnh `docker logs -f <container-id>` (`-f` để stream log trực tiếp).
 
![ld1](../../images/ld1.PNG)
 
#### Log trong Kubernetes
```
apiVersion: v1
kind: Pod
metadata:
  name: event-simulator-pod
spec:
  containers:
  - name: event-simulator
    image: kodekloud/event-simulator
```
 ![logs-k8s](../../images/logs-k8s.png)
 
- Để xem log
  ```
  $ kubectl logs -f event-simulator-pod
  ```
- Nếu Pod có nhiều container thì bạn phải chỉ định rõ tên container trong lệnh.
  ```
  $ kubectl logs -f <pod-name> <container-name>
  $ kubectl logs -f even-simulator-pod event-simulator
  ```

  ![logs1](../../images/logs1.PNG)

> 💡 **Bổ sung:** Các tùy chọn `kubectl logs` thường dùng khi thi và troubleshooting:
> - `kubectl logs <pod> -c <container>`: cách chỉ định container rõ ràng (tương đương truyền tên container làm tham số thứ hai). Nếu không chỉ định với Pod nhiều container, kubectl sẽ dùng container mặc định (annotation `kubectl.kubernetes.io/default-container`) hoặc container đầu tiên và in ra thông báo.
> - `kubectl logs <pod> --all-containers`: xem log của tất cả container trong Pod.
> - `kubectl logs <pod> --previous` (`-p`): xem log của lần chạy **trước** của container — rất hữu ích khi container bị `CrashLoopBackOff`.
> - `kubectl logs <pod> --tail=50`, `--since=1h`, `--timestamps`: giới hạn số dòng/khoảng thời gian, thêm timestamp.
> - `kubectl logs deploy/<tên-deployment>` hoặc `kubectl logs -l app=web`: xem log theo Deployment hoặc theo label selector.
>
> Về bản chất, container runtime ghi stdout/stderr của container vào file trên node (thường ở `/var/log/pods/` và `/var/log/containers/`); kubelet đọc các file này để phục vụ `kubectl logs`. Log của các thành phần chạy dưới dạng systemd service (vd kubelet, containerd) xem bằng `journalctl -u kubelet`. Kubernetes không có sẵn giải pháp lưu trữ log tập trung (cluster-level logging) — cần dùng thêm công cụ như Fluentd/Fluent Bit, Loki, Elastic Stack...
  
#### Tài liệu tham khảo K8s
- https://kubernetes.io/blog/2015/06/cluster-level-logging-with-kubernetes/
- https://kubernetes.io/docs/concepts/cluster-administration/logging/
 
