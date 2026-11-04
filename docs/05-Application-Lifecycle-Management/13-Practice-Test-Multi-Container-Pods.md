# Practice Test - Multi-Container Pods (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-multi-container-pods/)
  
Lời giải cho bài practice test - multi-container pods
- Xác định số lượng container đang chạy trong Pod 'red'.
  
  <details>
  
  ```
  $ kubectl get pod red
  ```

  Xem cột `READY` (vd `0/3` nghĩa là Pod có 3 container).
  </details>
  
- Xác định tên các container đang chạy trong Pod 'blue'.

  <details>
  ```
  $ kubectl describe pod blue
  ```

  > 💡 **Mẹo:** Lấy nhanh tên container: `kubectl get pod blue -o jsonpath='{.spec.containers[*].name}'`.
  </details>
    
- File đáp án nằm tại /var/answers/answer-yellow.yaml

  <details>
  ```
  $ kubectl create -f /var/answers/answer-yellow.yaml
  ```

  > 💡 **Gợi ý:** Tự tạo file mẫu bằng `kubectl run yellow --image=busybox --dry-run=client -o yaml > yellow.yaml`, sau đó thêm container thứ hai vào mảng `containers`.
  </details>
  
- Chúng tôi đã triển khai một logging stack cho ứng dụng trong namespace elastic-stack. Hãy kiểm tra nó.
  
  <details>
  ```
  $ kubectl get pods -n elastic-stack
  ```
  </details>
  
- Kiểm tra giao diện Kibana qua link phía trên terminal. Hiện tại chưa có log nào.

- Chạy `kubectl describe pod -n elastic-stack`

  <details>
  ```
  $ kubectl describe pod -n elastic-stack
  ```
  </details>
  
- Chạy lệnh 'kubectl -n elastic-stack exec -it app cat /log/app.log'
  
  <details>
  ```
  $ kubectl -n elastic-stack exec -it app cat /log/app.log
  ```

  > 💡 **Lưu ý:** Các phiên bản kubectl mới bắt buộc dùng `--` để ngăn cách lệnh cần chạy: `kubectl -n elastic-stack exec -it app -- cat /log/app.log`. Cũng có thể dùng `kubectl -n elastic-stack logs app`.
  </details>
  
- File đáp án nằm tại /var/answers/answer-app.yaml

  > 💡 **Giải thích:** Cần thêm một sidecar container (image `kodekloud/filebeat-configured`) vào Pod `app`, mount cùng volume chứa log (`/var/log/event-simulator/`) để đọc log và đẩy sang Elasticsearch. Vì không thể thêm container vào Pod đang chạy, hãy dùng `kubectl edit` + `kubectl replace --force -f <file>` để tạo lại Pod.
  
- Kiểm tra giao diện Kibana. Bây giờ bạn sẽ thấy log xuất hiện trong mục 'Discover'. Có thể phải đợi vài phút để log được đẩy lên. Có thể bạn cần tạo một index pattern để liệt kê log. Nếu chưa rõ, hãy xem video này: https://bit.ly/2EXYdHf
  
