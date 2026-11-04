# Practice Test - Xem Certificate (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-view-certificate-details/)
  
Lời giải cho bài practice test - xem certificate

> 💡 **Mẹo:** Hầu hết câu hỏi trong bài này đều trả lời được bằng cách đọc các file manifest static Pod trong `/etc/kubernetes/manifests/` (lưu ý thư mục là `manifests`, có chữ **s**). Dùng `grep` để tìm nhanh, ví dụ `grep -- --tls-cert-file /etc/kubernetes/manifests/kube-apiserver.yaml` hoặc `grep etcd /etc/kubernetes/manifests/kube-apiserver.yaml`.

- Xác định file certificate được dùng cho kube-apiserver
  
  <details>
  
  ```
  $ cat /etc/kubernetes/manifest/kube-apiserver.yaml
  
  Answer: /etc/kubernetes/pki/apiserver.crt
  ```
  </details>

  (Tìm tùy chọn `--tls-cert-file`.)
  
- Xác định file certificate mà kube-apiserver dùng để xác thực với tư cách client tới etcd server
  
  <details>
  ```
  $ cat /etc/kubernetes/manifest/kube-apiserver.yaml
  Answer: /etc/kubernetes/pki/apiserver-etcd-client.crt
  ```
  </details>

  (Tìm tùy chọn `--etcd-certfile`.)
  
- Tìm tùy chọn `--kubelet-client-key` trong file /etc/kubernetes/manifests/kube-apiserver.yaml
  
  <details>
  ```
  Answer: /etc/kubernetes/pki/apiserver-kubelet-client.key
  ```
  </details>
  
- Tìm tùy chọn `--cert-file` trong file /etc/kubernetes/manifests/etcd.yaml
  
  <details>
  ```
  Answer: /etc/kubernetes/pki/etcd/server.crt
  ```
  </details>
  
- Tìm CA certificate (tùy chọn `--trusted-ca-file`) trong file /etc/kubernetes/manifests/etcd.yaml
  
  <details>
  ```
  Answer: /etc/kubernetes/pki/etcd/ca.crt
  ```
  </details>
  
- Chạy lệnh openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text (xem CN trong mục Subject)
  
  <details>
  ```
  $ openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
  ```
  </details>
  
- Chạy lệnh openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text và tìm mục Issuer
  
  <details>
  ```
  $ openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text 
  ```
  </details>
  
- Chạy lệnh openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text và xem mục Alternative Names (Subject Alternative Name)
  
  <details>
  ```
  $ openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
  ```
  </details>
  
- Chạy lệnh openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text và tìm CN trong mục Subject.
  
  <details>
  ```
  $ openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text
  ```
  </details>
  
- Chạy lệnh openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text và kiểm tra ngày hết hạn (Not After).
  
  <details>
  ```
  $ openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
  ```
  </details>
  
- Chạy lệnh 'openssl x509 -in /etc/kubernetes/pki/ca.crt -text' và xem mục Validity
  
  <details>
  ```
  $ openssl x509 -in /etc/kubernetes/pki/ca.crt -text
  ```
  </details>

  (Theo mặc định kubeadm cấp CA có hạn 10 năm, còn các certificate thành phần có hạn 1 năm.)
  
- Kiểm tra tùy chọn `--cert-file` trong file manifest (đường dẫn certificate của etcd bị cấu hình sai, cần sửa lại cho đúng `/etc/kubernetes/pki/etcd/server.crt`).
  
  <details>
  ```
  $ vi /etc/kubernetes/manifests/etcd.yaml
  ```
  </details>
  
- etcd có CA riêng. Phải dùng đúng CA của etcd cho tùy chọn `--etcd-cafile` trong /etc/kubernetes/manifests/kube-apiserver.yaml (giá trị đúng là `/etc/kubernetes/pki/etcd/ca.crt`).
  
  <details>
  ```
  View answer at /var/answers/kube-apiserver.yaml
  ```
  </details>

> 💡 **Mẹo troubleshooting:** Khi kube-apiserver hoặc etcd chết do cấu hình certificate sai, `kubectl` sẽ không dùng được. Hãy dùng `crictl ps -a | grep kube-apiserver` (hoặc `etcd`) rồi `crictl logs <container-id>` để đọc lỗi, hoặc xem log trong `/var/log/pods/`. Sau khi sửa file manifest, kubelet sẽ tự tạo lại static Pod — đợi một lúc rồi chạy lại `kubectl get pods -n kube-system`.
