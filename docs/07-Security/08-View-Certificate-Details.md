# Xem chi tiết Certificate
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/view-certificate-details/)
  
Trong phần này, chúng ta sẽ tìm hiểu cách xem các certificate trong một Kubernetes cluster.

## Xem certificate
- Trước tiên cần biết cluster được dựng theo cách nào: cài thủ công từ đầu ("the hard way" — các thành phần chạy dưới dạng service của hệ điều hành) hay bằng kubeadm (các thành phần chạy dưới dạng static Pod). Từ đó tìm xem certificate được khai báo ở đâu.

 ![hrd](../../images/hrd.PNG)

 ![hrd1](../../images/hrd1.PNG)
 
 - Để xem chi tiết một certificate
   ```
   $ openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
   ```
   
   ![hrd2](../../images/hrd2.PNG)

> 💡 **Bổ sung:** Các trường cần chú ý trong output: **Subject** (CN = tên của certificate), **Issuer** (CA đã ký), **Validity / Not After** (ngày hết hạn), và **X509v3 Subject Alternative Name** (các tên/IP thay thế). Với cluster kubeadm, có thể kiểm tra nhanh hạn của tất cả certificate bằng `kubeadm certs check-expiration` và gia hạn bằng `kubeadm certs renew all` (sau đó cần restart các static Pod của control plane).
   
#### Làm tương tự để lấy thông tin của tất cả các certificate khác

   ![hrd3](../../images/hrd3.PNG)
   
## Kiểm tra log - Cluster cài thủ công
- Kiểm tra log của service bằng journalctl
  ```
  $ journalctl -u etcd.service -l
  ```
  
  ![hrd4](../../images/hrd4.PNG)
  
## Kiểm tra log - Cluster dựng bằng kubeadm
- Xem log bằng kubectl
  ```
  $ kubectl logs etcd-master
  ```
  ![hrd5](../../images/hrd5.PNG)
  
- Nếu kube-apiserver hoặc etcd bị lỗi khiến kubectl không hoạt động, xem log bằng docker ps và docker logs
  ```
  $ docker ps -a
  $ docker logs <container-id>
  ```
  ![hrd6](../../images/hrd6.PNG)

> 💡 **Bổ sung:** Dockershim đã bị gỡ khỏi Kubernetes từ v1.24, nên các cluster hiện nay (kể cả môi trường thi CKA) dùng containerd/CRI-O và **không có lệnh `docker`**. Thay vào đó dùng `crictl`: `crictl ps -a` để liệt kê container (kể cả container đã chết) và `crictl logs <container-id>` để xem log. Ngoài ra có thể xem trực tiếp file log trong `/var/log/pods/` hoặc `/var/log/containers/`, và log của kubelet bằng `journalctl -u kubelet`.
  
#### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/setup/best-practices/certificates/#certificate-paths
