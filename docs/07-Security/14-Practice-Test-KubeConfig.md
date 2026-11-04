# Practice Test - KubeConfig (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-kubeconfig/)
 
Lời giải cho bài practice test - kubeconfig
- Tìm file kubeconfig trong thư mục `/root/.kube`
  
  <details>
  ```
  $ ls -l /root/.kube
  ```
  </details>
    
- Chạy lệnh kubectl config view và đếm số cluster

  <details>
  ```
  $ kubectl config view
  ```
  </details>
    
- Chạy lệnh `kubectl config view` và đếm số user

  <details>
  ```
  $ kubectl config view
  ```
  </details>
  
- Có bao nhiêu context được định nghĩa trong file kubeconfig mặc định?
  
  <details>
  ```
  $ kubectl config view
  ```
  </details>

  (Có thể dùng `kubectl config get-contexts` để xem nhanh.)
  
- Chạy lệnh 'kubectl config view' và tìm tên user.
  
  <details>
  ```
  $ kubectl config view
  ```
  </details>
  
- Tên của cluster được cấu hình trong file kubeconfig mặc định là gì?
  
  <details>
  ```
  $ kubectl config view
  ```
  </details>
  
- Chạy lệnh 'kubectl config view --kubeconfig my-kube-config'
  
  <details>
  ```
  $ kubectl config view --kubeconfig my-kube-config
  ```
  </details>
  
- Có bao nhiêu context được cấu hình trong file 'my-kube-config'?
  
  <details>
  ```
  $ kubectl config view --kubeconfig my-kube-config
  ```
  </details>
  
- User nào được cấu hình trong context 'research'?
  
  <details>
  ```
  $ kubectl config view --kubeconfig my-kube-config
  ```
  </details>
    
- Tên file client-certificate được cấu hình cho 'aws-user' là gì?
  
  <details>
  ```
  $ kubectl config view --kubeconfig my-kube-config
  ```
  </details>
  
- Current context trong file 'my-kube-config' đang là gì?
  
  <details>
  ```
  $ kubectl config view --kubeconfig my-kube-config
  ```
  </details>

  (Hoặc dùng `kubectl config current-context --kubeconfig my-kube-config`.)
  
- Chạy lệnh kubectl config --kubeconfig=/root/my-kube-config use-context research
  
  <details>
  ```
  $ kubectl config --kubeconfig=/root/my-kube-config use-context research
  ```
  </details>
  
- Thay thế nội dung file kubeconfig mặc định bằng nội dung của file my-kube-config.
  
  <details>
  ```
  $ mv .kube/config .kube/config.bak
  $ cp /root/my-kube-config .kube/config
  ```
  </details>

  (Cách khác không cần ghi đè file: `export KUBECONFIG=/root/my-kube-config`. Tuy nhiên biến môi trường chỉ có hiệu lực trong phiên shell hiện tại, trừ khi được thêm vào `~/.bashrc`.)
  
- Đường dẫn tới certificate trong file kubeconfig bị sai. Hãy sửa lại. Tất cả certificate của user được lưu tại /etc/kubernetes/pki/users
  
 <details>
  $ kubectl get pods
  master $ ls
  dev-user.crt  dev-user.csr  dev-user.key
  master $ vi /root/.kube/config
  master $ grep dev-user.crt /root/.kube/config
    client-certificate: /etc/kubernetes/pki/users/dev-user/dev-user.crt
  master $ pwd
  /etc/kubernetes/pki/users/dev-user
  master $ kubectl get pods
  No resources found in default namespace.
 </details>

> 💡 **Giải thích:** Khi `kubectl get pods` báo lỗi kiểu `unable to read client-cert ... no such file or directory`, hãy liệt kê thư mục `/etc/kubernetes/pki/users/<user>/` để tìm đúng tên file, rồi sửa trường `client-certificate` (và `client-key` nếu cần) trong `/root/.kube/config` cho khớp. Trong bài lab, file đúng là `dev-user.crt` thay vì tên bị gõ sai trong kubeconfig.
