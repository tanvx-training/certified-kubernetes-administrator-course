# Practice Test - Certificates API (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-certificates-api/)

Lời giải cho bài practice test - Certificates API
- Một thành viên mới tên akshay vừa gia nhập team và cần quyền truy cập vào cluster. File Certificate Signing Request nằm ở thư mục /root.

  <details>
  ```
  $ ls -l /root
  ```
  </details>
  
- Xem đáp án tại /var/answers/akshay-csr.yaml
  
  <details>
  ```
  $ kubectl create -f /var/answers/akshay-csr.yaml
  ```
  </details>

> 💡 **Bổ sung:** Nếu phải tự viết file YAML, dùng `apiVersion: certificates.k8s.io/v1`, `signerName: kubernetes.io/kube-apiserver-client`, `usages: [client auth]`, và trường `request` lấy từ `cat /root/akshay.csr | base64 -w 0`.
  
- Chạy lệnh kubectl get csr (kiểm tra trạng thái của CSR vừa tạo — lúc này là `Pending`)
  
  <details>
  ```
  $ kubectl get csr
  ```
  </details>
    
- Chạy lệnh kubectl certificate approve akshay
  
  <details>
  ```
  $ kubectl certificate approve akshay
  ```
  </details>
  
- Chạy lệnh kubectl get csr (trạng thái chuyển thành `Approved,Issued`)
  
  <details>
  ```
  $ kubectl get csr
  ```
  </details>
  
- Chạy lệnh kubectl get csr và xem cột Requestor
  
  <details>
  ```
  $ kubectl get csr
  ```
  </details>
  
- Các CSR còn lại được tạo ra trong quá trình TLS Bootstrapping (khi kubelet trên node tham gia cluster và xin certificate). Chúng ta sẽ tìm hiểu kỹ hơn ở phần TLS bootstrap sau trong khóa học.

- Chạy lệnh kubectl get csr
  
  <details>
  ```
  $ kubectl get csr
  ```
  </details>
  
- Chạy lệnh kubectl get csr agent-smith -o yaml (xem trường `groups` — request này xin quyền `system:masters`, tức là toàn quyền admin, nên không được duyệt)
  
  <details>
  ```
  $ kubectl get csr agent-smith -o yaml
  ```
  </details>
  
- Chạy lệnh kubectl certificate deny agent-smith
  
  <details>
  ```
  $ kubectl certificate deny agent-smith
  ```
  </details>
  
- Chạy lệnh kubectl delete csr agent-smith
  
  <details>
  ```
  $ kubectl delete csr agent-smith
  ```
  </details>
  
