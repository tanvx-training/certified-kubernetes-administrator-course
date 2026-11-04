# Các phiên bản phần mềm Kubernetes
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/kubernetes-software-versions/)
  
Trong phần này, chúng ta sẽ tìm hiểu về các bản phát hành (release) và phiên bản (version) của Kubernetes.

#### Xem phiên bản Kubernetes đã cài đặt
```
$ kubectl get nodes
```
![kgn](../../images/kgn.PNG)

#### Phân tích kỹ hơn số phiên bản
- Gồm 3 phần
  - Phần đầu là major version (phiên bản chính)
  - Phần thứ hai là minor version (phiên bản phụ) – thêm tính năng mới
  - Cuối cùng là patch version (bản vá) – sửa lỗi
  
  ![mmp](../../images/mmp.PNG)
  
#### Kubernetes tuân theo quy trình đánh số phiên bản phát hành chuẩn (semantic versioning)
- Bạn có thể xem tất cả các bản phát hành Kubernetes tại https://github.com/kubernetes/kubernetes/releases

  ![r1](../../images/r1.PNG)
  
  ![r2](../../images/r2.PNG)

> 💡 **Bổ sung:**
> - Ngoài bản stable, Kubernetes còn có các bản pre-release như `alpha` và `beta` (vd `v1.31.0-alpha.1`, `v1.31.0-beta.0`, `v1.31.0-rc.0`). Tính năng mới thường đi qua các giai đoạn alpha (mặc định tắt, dùng feature gate) → beta → GA (stable).
> - Hiện nay Kubernetes phát hành khoảng **3 minor version mỗi năm** (khoảng 4 tháng một bản), và mỗi minor version được hỗ trợ bản vá khoảng **14 tháng** (12 tháng hỗ trợ chính + khoảng 2 tháng giai đoạn upgrade). Thực tế dự án duy trì 3 minor version gần nhất.
> - Lệnh hữu ích: `kubectl version` (hiển thị phiên bản client và server), `kubeadm version`, `kubelet --version`.
  
#### Gói tải về chứa tất cả các thành phần Kubernetes, ngoại trừ **`ETCD Cluster`** và **`CoreDNS`** vì đây là các dự án riêng biệt.

 ![r3](../../images/r3.PNG)

> 💡 **Bổ sung:** Các thành phần control plane (kube-apiserver, kube-controller-manager, kube-scheduler), kubelet, kube-proxy cùng chung số phiên bản với release Kubernetes. etcd và CoreDNS có hệ thống đánh số phiên bản riêng (vd etcd `3.5.x`, CoreDNS `1.11.x`); mỗi bản Kubernetes sẽ khuyến nghị một phiên bản etcd/CoreDNS cụ thể, và `kubeadm upgrade` sẽ tự upgrade chúng theo.
 
#### Tài liệu tham khảo

 - https://blog.risingstack.com/the-history-of-kubernetes/
 - https://kubernetes.io/docs/setup/release/version-skew-policy/
 
 






