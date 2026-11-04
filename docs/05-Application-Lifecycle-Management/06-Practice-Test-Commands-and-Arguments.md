# Practice Test - Commands and Arguments (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-commands-and-arguments/)
  
Lời giải cho bài practice test - commands and arguments
- Chạy lệnh 'kubectl get pods' và đếm số lượng Pod.
  
  <details>
  
  ```
  $ kubectl get pods
  ```
  </details>
  
- Chạy lệnh 'kubectl describe pod' và tìm tùy chọn command.

  <details>
  
  ```
  $ kubectl describe pod
  ```
  </details>
  
- Đặt tùy chọn command thành ['sleep', '5000']. File đáp án tại: /var/answers/answer-ubuntu-sleeper-2.yaml

  > 💡 **Gợi ý:** Có thể tạo nhanh bằng `kubectl run ubuntu-sleeper-2 --image=ubuntu --dry-run=client -o yaml --command -- sleep 5000 > pod.yaml` rồi `kubectl apply -f pod.yaml`.

- Cả sleep và 1200 đều phải được định nghĩa dưới dạng chuỗi (string). File đáp án tại: /var/answers/answer-ubuntu-sleeper-3.yaml

  > 💡 **Giải thích:** Nếu viết `command: ["sleep", 1200]`, YAML hiểu `1200` là số nguyên và API server sẽ báo lỗi vì `command` phải là mảng các string. Hãy viết `"1200"`.

- File đáp án tại: /var/answers/answer-ubuntu-sleeper-3-2.yaml

  > 💡 **Giải thích:** Câu này yêu cầu sửa giá trị sleep của Pod đang chạy. Vì không thể thay đổi `command` của Pod đang chạy, cần xóa Pod và tạo lại: `kubectl edit pod ubuntu-sleeper-3` (sửa rồi lưu → bị từ chối, bản sửa được lưu vào `/tmp/`), sau đó `kubectl replace --force -f /tmp/kubectl-edit-xxxx.yaml`.

- Xem file 'Dockerfile' tại /root/webapp-color. Command nào được chạy khi container khởi động?
  
  <details>
  
  ```
  python app.py
  ```
  </details>
  
- Xem file 'Dockerfile2' tại /root/webapp-color. Command nào được chạy khi container khởi động?

  <details>
  ```
  python app.py --color red
  ```

  > 💡 **Giải thích:** Dockerfile2 có `ENTRYPOINT ["python", "app.py"]` và `CMD ["--color", "red"]`, nên CMD được nối vào sau ENTRYPOINT.
  </details>
  
- 'command' (entrypoint) bị ghi đè trong file định nghĩa Pod. Vì vậy đáp án là --color green

- Xem hai file trong thư mục 'webapp-color-3'. Command nào được chạy khi container khởi động?

  <details>
  
  ```
  python app.py --color pink
  ```

  > 💡 **Giải thích:** Trong file định nghĩa Pod, `command` ghi đè ENTRYPOINT của Dockerfile và `args` ghi đè CMD. Hãy kết hợp giá trị sau khi ghi đè: `command` + `args`.
  </details>
  
- File đáp án tại /var/answers/answer-webapp-color-green.yaml

  > 💡 **Gợi ý:** Truyền argument `--color green` cho container mà không đổi command: `kubectl run webapp-green --image=kodekloud/webapp-color -- --color green`.
