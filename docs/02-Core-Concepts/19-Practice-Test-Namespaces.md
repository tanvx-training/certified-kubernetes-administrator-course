# Practice Test - Namespaces (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-namespaces/)

Lời giải cho bài practice test Namespaces

1. <details>
   <summary>Có bao nhiêu Namespace trên hệ thống?</summary>

   ```
   kubectl get namespace
   ```

   Đếm số namespace hiển thị (nếu có).

   > 💡 **Bổ sung:** Có thể để hệ thống tự đếm: `kubectl get ns --no-headers | wc -l`.
   </details>

1. <details>
   <summary>Có bao nhiêu Pod trong namespace research?</summary>

   ```
   kubectl get pods --namespace=research
   ```

   Đếm số Pod hiển thị (nếu có).
   </details>

1. <details>
   <summary>Tạo một Pod trong namespace finance.</summary>

   ```
   kubectl run redis --image=redis --namespace=finance
   ```
   </details>

1. <details>
   <summary>Pod blue nằm trong namespace nào?</summary>

   ```
   kubectl get pods --all-namespaces
   ```

   Xem kết quả output.

   Hoặc dùng `grep` để lọc kết quả - lưu ý `NAMESPACE` là cột đầu tiên của output

   ```
   kubectl get pods --all-namespaces | grep blue
   ```

   </details>

1. <details>
   <summary>Truy cập ứng dụng web Blue bằng đường link phía trên terminal!!</summary>

   Nhấn nút `blue-application-ui` ở phía trên terminal. Thử nhập như sau:

   ```
   Host Name: db-service
   Host Port: 6379
   ```
   </details>

1. <details>
   <summary>Ứng dụng Blue nên dùng tên DNS nào để truy cập database db-service trong chính namespace của nó - marketing?</summary>

   > db-service

   Để truy cập Service trong cùng namespace, chỉ cần phần host name của tên miền đầy đủ (FQDN - fully qualified domain name).

   </details>

1. <details>
   <summary>Ứng dụng Blue nên dùng tên DNS nào để truy cập database db-service trong namespace dev?</summary>

   Hoặc dùng FQDN

   > db-service.dev.svc.cluster.local

   Hoặc chỉ cần thêm tên namespace vào sau

   > db-service.dev

   > 💡 **Bổ sung:** FQDN của một Service có dạng `<tên-service>.<namespace>.svc.<cluster-domain>`, với cluster domain mặc định là `cluster.local`. Tên ngắn hoạt động được là nhờ file `/etc/resolv.conf` trong Pod có dòng `search <namespace>.svc.cluster.local svc.cluster.local cluster.local`, nên CoreDNS sẽ tự ghép thêm phần đuôi. Vì vậy `db-service` chỉ phân giải được Service **cùng namespace** với Pod, còn sang namespace khác phải có ít nhất `db-service.dev`.

   </details>

