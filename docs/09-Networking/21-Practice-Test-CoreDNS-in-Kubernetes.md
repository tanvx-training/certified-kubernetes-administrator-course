# Practice Test CoreDNS in Kubernetes (Bài thực hành CoreDNS trong Kubernetes)

  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-coredns-in-kubernetes/)

#### Lời giải

  1. Xem lời giải

     <details>

      ```
      CoreDNS
      ```
     </details>
  
  2. Xem lời giải

     <details>

      ```
      2
      ```
     </details>

  3. Xem lời giải

     <details>

      ```
      10.96.0.10
      ```

      > 💡 Đây là ClusterIP của Service `kube-dns` trong namespace `kube-system`: `kubectl get svc -n kube-system`.
     </details>

  4. Xem lời giải

     <details>

      ```
      /etc/coredns/Corefile

      HOẶC

      kubectl -n kube-system describe deployments.apps coredns | grep -A2 Args | grep Corefile
      ```

      > 💡 File Corefile được truyền cho CoreDNS qua tham số `-conf /etc/coredns/Corefile` trong Deployment `coredns`, và nội dung của nó được mount từ ConfigMap `coredns`.
     </details>

  5. Xem lời giải

     <details>

      ```
      Configured as a ConfigMapObject
      ```
     </details>

  6. Xem lời giải

     <details>

      ```
      CoreDNS
      ```
     </details>

  7. Xem lời giải

     <details>

      ```
      coredns
      ```
     </details>

  8. Xem lời giải

     <details>

      ```
      cluster.local
      ```

      > 💡 Domain gốc của cluster nằm trong dòng `kubernetes cluster.local in-addr.arpa ip6.arpa` của Corefile: `kubectl -n kube-system describe cm coredns`.
     </details>

  9. Xem lời giải

     <details>

      ```
      Ok
      ```
     </details>

  10. Xem lời giải

      <details>

       ```
       web-service
       ```
      </details>

  11. Xem lời giải

      <details>
 
       ```
       web-service.default.pod
       ```

       > 💡 Tên này không dùng được vì bản ghi Service phải nằm dưới `svc`, còn bản ghi dưới `pod` có dạng `<IP-có-gạch-ngang>.<namespace>.pod.cluster.local`, không phải tên Service.
      </details>

  12. Xem lời giải

      <details>
 
       ```
       web-service.payroll
       ```

       > 💡 Service nằm ở namespace khác (`payroll`) nên phải thêm tên namespace sau tên Service; chỉ gọi `web-service` sẽ được ghép với namespace hiện tại của Pod (theo dòng `search` trong `/etc/resolv.conf`).
      </details>

  13. Xem lời giải

      <details>
 
       ```
       web-service.payroll.svc.cluster
       ```

       > 💡 Tên này không phân giải được vì `svc.cluster` không phải là hậu tố hợp lệ: phải là `web-service.payroll.svc` hoặc FQDN đầy đủ `web-service.payroll.svc.cluster.local`.
      </details>

  14. Xem lời giải

      <details>
 
       ```
       kubectl edit deploy webapp
 
       Tìm biến DB_Host và đổi giá trị DB_Host từ mysql thành mysql.payroll
 
       spec:
         containers:
         - env:
           - name: DB_Host
             value: mysql.payroll
       ```

       > 💡 Sửa Deployment sẽ tạo ra Pod mới với biến môi trường mới (rolling update). Mẹo: có thể làm nhanh bằng `kubectl set env deploy/webapp DB_Host=mysql.payroll`.
      </details>
 
  15. Xem lời giải

      <details>
 
       ```
       kubectl exec -it hr -- nslookup mysql.payroll > /root/nslookup.out
       ```

       > 💡 Lệnh `nslookup` chạy bên trong Pod `hr`, còn dấu `>` được shell trên máy controlplane xử lý, nên file `/root/nslookup.out` được ghi trên controlplane (đúng như đề yêu cầu). Khi redirect output ra file, nên bỏ `-t` (dùng `kubectl exec hr -- ...`) để tránh ký tự xuống dòng `\r` của TTY lẫn vào file.
      </details>