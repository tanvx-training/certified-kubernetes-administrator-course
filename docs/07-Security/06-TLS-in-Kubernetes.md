# TLS trong Kubernetes
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/tls-in-kubernetes/)
  
Trong phần này, chúng ta sẽ tìm hiểu cách TLS được sử dụng trong Kubernetes.

#### Hai yêu cầu chính là: mọi service trong cluster phải dùng server certificate, và mọi client phải dùng client certificate để chứng minh danh tính của mình.
- Server Certificate cho các server
- Client Certificate cho các client

  ![tls](../../images/tls.PNG)
  
#### Hãy cùng xem các thành phần trong K8s cluster, xác định đâu là server, đâu là client và thành phần nào giao tiếp với thành phần nào.

  ![certs](../../images/certs.PNG)

> 💡 **Bổ sung:** Tóm tắt các cặp server/client chính:
> - **Server:** kube-apiserver (`apiserver.crt/.key`), etcd (`etcd/server.crt/.key`), kubelet trên mỗi node (`kubelet.crt/.key`, phục vụ API của kubelet ở cổng 10250).
> - **Client của kube-apiserver:** admin (kubectl), kube-scheduler, kube-controller-manager, kube-proxy, kubelet — mỗi thành phần có client certificate riêng.
> - **kube-apiserver cũng là client:** khi nói chuyện với etcd (`apiserver-etcd-client.crt`) và với kubelet (`apiserver-kubelet-client.crt`).
> - Cần ít nhất một CA để ký các certificate này; kubeadm dùng CA riêng cho etcd (`/etc/kubernetes/pki/etcd/ca.crt`) tách biệt với CA của cluster (`/etc/kubernetes/pki/ca.crt`).
