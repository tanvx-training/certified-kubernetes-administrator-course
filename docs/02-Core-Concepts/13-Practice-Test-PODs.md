# Practice Test - Pods (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-pods/)

## Lời giải cho bài practice test
1. <details>
   <summary>Có bao nhiêu Pod đang tồn tại trên hệ thống?</summary>

   ```bash
   kubectl get pods
   ```

   Đếm số Pod hiển thị (nếu có).

   > 💡 **Bổ sung:** `kubectl get pods` chỉ liệt kê Pod trong namespace hiện tại (mặc định là `default`). Muốn xem Pod ở mọi namespace, dùng `kubectl get pods -A` (viết tắt của `--all-namespaces`).

   </details>

1. <details>
   <summary>Tạo một Pod mới với image nginx.</summary>

   ```bash
   kubectl run nginx --image=nginx
   ```

   > 💡 **Bổ sung:** Từ Kubernetes v1.18, `kubectl run` chỉ tạo Pod (flag `--generator` đã bị loại bỏ). Muốn tạo Deployment, dùng `kubectl create deployment`.

   </details>

1. <details>
   <summary>Bây giờ có bao nhiêu Pod đã được tạo?</summary>

   ```bash
   kubectl get pods
   ```

   Đếm số Pod hiển thị (nếu có).

   Để hệ thống tự đếm giúp bạn, có thể dùng lệnh sau:

   ```bash
   kubectl get pods --no-headers | wc -l
   ```

   * `--no-headers` thì khá rõ nghĩa - chỉ in ra phần dữ liệu, bỏ dòng tiêu đề.
   * `wc` là chương trình đếm từ (word count). `-l` yêu cầu nó đếm số dòng thay vì số từ, tức là đếm số dòng mà `kubectl` in ra.
   </details>

1. <details>
   <summary>Image nào được dùng để tạo các Pod mới?</summary>

    `kubectl describe` in ra rất nhiều thông tin. Lệnh dưới đây sẽ describe tất cả các Pod có tên bắt đầu bằng `newpods`, sau đó lọc bằng `grep` để lấy đúng thông tin cần tìm.

    ```bash
    kubectl describe pod newpods | grep image
    ```

    Ta thấy cả ba Pod đều pull cùng một image.

    > 💡 **Bổ sung:** `grep` phân biệt hoa thường, và dòng `Image:` trong output viết hoa chữ I. Lệnh trên vẫn cho kết quả nhờ khớp dòng event `Pulling image "..."`. Để chắc chắn hơn, dùng `grep -i image`. Một cách khác gọn hơn: `kubectl get pods -o wide` không hiện image, nhưng `kubectl get pods -o jsonpath='{.items[*].spec.containers[*].image}'` sẽ in ra image của mọi Pod.

   </details>

1. <details>
   <summary>Các Pod này được đặt trên những node nào?</summary>

    ```bash
    kubectl get pods -o wide
    ```

    Xem cột `NODE` của từng Pod trong 3 Pod `newpods`.

   </details>

1. <details>
   <summary>Pod webapp có bao nhiêu container?</summary>

   ```bash
   kubectl describe pod webapp
   ```

   Xem trong mục `Containers`. Để ý thấy có `nginx` và `agentx`.

   </details>

1. <details>
   <summary>Những image nào được dùng trong Pod webapp mới?</summary>

   Xem lại output của câu 6. Với mỗi container đã tìm thấy, xem dòng `Image:`.

   </details>

1. <details>
   <summary>Trạng thái của container agentx trong Pod webapp là gì?</summary>

   ```bash
   kubectl describe pod webapp
   ```

   Xem trường `State:` của container `agentx`.

   </details>

1. <details>
   <summary>Theo bạn, vì sao container agentx trong Pod webapp bị lỗi?</summary>

   Xem lại output của câu 8 và nhìn vào mục `Events:` ở cuối. Tìm event có nội dung `failed to pull and unpack image ...`

   > 💡 **Bổ sung:** Đây là lỗi pull image điển hình (thường do sai tên image/tag, hoặc registry yêu cầu xác thực). Trạng thái Pod khi đó sẽ là `ErrImagePull` rồi chuyển sang `ImagePullBackOff` - kubelet sẽ thử pull lại với thời gian chờ tăng dần.

   </details>

1. <details>
   <summary>Cột READY trong output của lệnh kubectl get pods biểu thị điều gì?</summary>

   ```bash
   kubectl get pods
   ```

   Nhìn vào Pod `webapp` - ta biết nó có hai container và một container đang lỗi. Từ đó bạn có thể suy ra đáp án đúng.

   > 💡 **Bổ sung:** `READY` có dạng `<số container đang ready>/<tổng số container trong Pod>`. Với `webapp`, giá trị sẽ là `1/2`.

   </details>

1. <details>
   <summary>Xóa Pod webapp.</summary>

   ```bash
   kubectl delete pod webapp
   ```

   Để xóa Pod ngay lập tức mà không phải chờ, có thể thêm flag `--force`. Lưu ý rằng trên hệ thống production thực tế, ép xóa là biện pháp cuối cùng vì nó có thể để lại các container chưa được dọn dẹp đúng cách. Một số ứng dụng có tác vụ dọn dẹp quan trọng cần chạy khi được yêu cầu dừng, và các tác vụ đó sẽ không được thực hiện.

   Tuy nhiên, trong kỳ thi bạn có thể dùng `--force` để tiết kiệm thời gian. Không có Pod nào trong đề thi phụ thuộc vào những điều trên.

   > 💡 **Bổ sung:** Cách hay dùng trong kỳ thi là `kubectl delete pod webapp --force --grace-period=0`. Có thể đặt sẵn biến môi trường `export now="--force --grace-period=0"` rồi gõ `kubectl delete pod webapp $now`.

   </details>

1. <details>
   <summary>Tạo một Pod mới tên redis với image redis123.</br>Sử dụng file YAML định nghĩa Pod.</summary>

   Để tạo file YAML định nghĩa Pod:

   `--dry-run=client` yêu cầu `kubectl` chạy thử mà không thực sự tạo gì cả. `-o yaml` nghĩa là "In ra console nội dung sẽ được gửi tới API server", sau đó ta chuyển hướng (redirect) output vào file chỉ định.

   ```bash
   kubectl run redis --image=redis123 --dry-run=client -o yaml > redis.yaml
   ```

   Giờ dùng file YAML vừa tạo để deploy Pod.

   ```bash
   kubectl create -f redis.yaml
   ```

   </details>

1. <details>
   <summary>Giờ hãy đổi image của Pod này thành redis.</br>Sau khi hoàn tất, Pod phải ở trạng thái running.</summary>

   Có ba cách để làm việc này!

   1. Cách 1</br>
      Sửa file manifest đã tạo ở câu trước

      ```bash
      vi redis.yaml
      ```

      Sửa tên image trong redis.yaml thành `redis`, lưu và thoát.

      Apply file yaml đã sửa

      ```bash
      kubectl apply -f redis.yaml
      ```

   1. Cách 2</br>
      Sửa trực tiếp Pod đang chạy (lưu ý không phải trường nào cũng sửa được theo cách này)

      ```
      kubectl edit pod redis
      ```

      Lệnh này sẽ mở YAML của Pod trong `vi`. Sửa giống như cách 1. Khi bạn thoát `vi`, thay đổi sẽ được áp dụng ngay. Nếu bạn sửa sai, bạn sẽ được đưa trở lại `vi`.

   1. Cách 3</br>
      Patch image trực tiếp. Với cách này bạn cần biết `name` của container trong Pod, vì ta gán image mới cho tên đó theo dạng `container_name=new_image`

      ```bash
      kubectl set image pod/redis redis=redis
      ```

   > 💡 **Bổ sung:** Với một Pod đang chạy, chỉ một số ít trường được phép sửa trực tiếp, chủ yếu là `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds` và thêm `spec.tolerations`. Muốn sửa các trường khác (ví dụ env, ports, command...), `kubectl edit` sẽ từ chối và lưu bản sửa vào một file tạm trong `/tmp/`; khi đó dùng `kubectl replace --force -f /tmp/kubectl-edit-xxxx.yaml` để xóa và tạo lại Pod.

   </details>
