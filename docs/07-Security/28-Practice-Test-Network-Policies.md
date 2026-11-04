# Practice Test - Network Policies (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-network-policies/)

Lời giải cho bài practice test - Network Policies

- Chạy lệnh 'kubectl get networkpolicy'
  
  <details>
  
  ```
  $ kubectl get networkpolicy
  ```
  
  </details>
  
- Chạy lệnh 'kubectl get networkpolicy'
   
  <details>
  
  ```
  $ kubectl get networkpolicy
  ```
  
  </details>
  
- Chạy lệnh 'kubectl get networkpolicy' và xem cột pod selector
  
  <details>
  
  ```
  $ kubectl get networkpolicy
  ```
  
  </details>

  > 💡 Sau đó dùng `kubectl get pods --show-labels` (hoặc `kubectl get pods -l <label>`) để tìm Pod khớp với pod selector đó.
  
- Chạy lệnh 'kubectl describe networkpolicy' và xem mục PolicyTypes
  
  <details>
  
  ```
  $ kubectl describe networkpolicy
  ```
  
  </details>
  
- Rule được cấu hình trên Network Policy này có tác động gì?
  
  <details>
  
  ```
  Traffic from internal to payroll pod is blocked
  ```
  
  </details>

  > 💡 Giải thích: policy `payroll-policy` chọn Pod payroll và chỉ cho phép ingress từ các Pod có label `name=internal` tới TCP port 8080; mọi traffic khác đi vào payroll đều bị chặn. Lưu ý: đáp án gốc ghi "blocked", nhưng theo cấu hình trên thì traffic từ internal tới payroll (port 8080) là **được phép** — hãy đối chiếu với các lựa chọn thực tế trong lab (thường đáp án đúng là "Traffic from Internal to Payroll POD is allowed").
  
- Rule được cấu hình trên Network Policy này có tác động gì?
  
  <details>
  
  ```
  Internal pod can access port 8080 on payroll pod
  ```
  
  </details>

  > 💡 Đáp án: Pod internal có thể truy cập port 8080 của Pod payroll.
  
- Truy cập giao diện của các ứng dụng này thông qua link ở phía trên terminal.

- Chỉ các ứng dụng internal mới có thể truy cập Service payroll.

- Thực hiện kiểm tra kết nối từ giao diện của ứng dụng Internal tới 'external-service' tại port '8080'.
  
  <details>
  
  ```
  Successful
  ```
  
  </details>

  > 💡 Đáp án: thành công — vì chưa có Network Policy nào chọn Pod internal nên egress của nó không bị hạn chế.
  
- File đáp án nằm tại /var/answers/answer-internal-policy.yaml
  
  <details>
  
  ```
  $ kubectl create -f /var/answers/answer-internal-policy.yaml
  ```
  
  </details>

  > 💡 Yêu cầu: tạo policy `internal-policy` cho Pod `internal` chỉ cho phép egress tới `payroll` (port 8080) và `mysql` (port 3306). Policy có `podSelector` chọn `name: internal`, `policyTypes: [Egress]`, và hai phần tử trong `egress`, mỗi phần tử gồm một `to.podSelector` (`name: payroll` / `name: mysql`) cùng `ports` tương ứng. Lưu ý: nếu Pod cần phân giải tên Service qua DNS, phải cho phép thêm egress port 53 (TCP/UDP).
  
  
