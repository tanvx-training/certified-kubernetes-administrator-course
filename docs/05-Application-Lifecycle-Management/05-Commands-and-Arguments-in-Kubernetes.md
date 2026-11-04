# Commands và Arguments trong Kubernetes
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/commands-and-arguments-in-kubernetes-2/)

Trong phần này, chúng ta sẽ tìm hiểu về command và argument trong Kubernetes.

- Bất cứ thứ gì được nối thêm vào cuối lệnh docker run sẽ tương ứng với thuộc tính **`args`** trong file định nghĩa Pod, dưới dạng một mảng.
- Trường **`command`** tương ứng với chỉ thị ENTRYPOINT trong Dockerfile. Tóm lại, có 2 trường tương ứng với 2 chỉ thị trong Dockerfile:
  - `command` ghi đè `ENTRYPOINT`
  - `args` ghi đè `CMD`
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: ubuntu-sleeper-pod
  spec:
   containers:
   - name: ubuntu-sleeper
     image: ubuntu-sleeper
     command: ["sleep2.0"]
     args: ["10"]
  ```
  ![args](../../images/args.PNG)

> 💡 **Bổ sung:**
> - Lưu ý tên dễ gây nhầm lẫn: trường `command` của Kubernetes **không** tương ứng với `CMD` của Docker mà tương ứng với `ENTRYPOINT`.
> - Mỗi phần tử trong `command`/`args` phải là **chuỗi (string)**. Số như `10` nên đặt trong dấu nháy (`"10"`), nếu không YAML sẽ hiểu là số nguyên và API server báo lỗi.
> - Có thể viết dạng mảng nhiều dòng, ví dụ `command:` rồi các dòng `- "sleep"`, `- "5000"`; hoặc dạng `command: ["sleep", "5000"]`.
> - Không thể sửa `command`/`args` của một Pod đang chạy (phần lớn spec của Pod là bất biến). Cách làm khi thi: `kubectl edit pod <tên>` → lưu thất bại, kubectl sẽ lưu bản chỉnh sửa vào file tạm trong `/tmp/` → chạy `kubectl replace --force -f /tmp/kubectl-edit-xxxx.yaml` để xóa và tạo lại Pod.
> - Tạo Pod nhanh với command: `kubectl run ubuntu-sleeper --image=ubuntu --command -- sleep 5000` (có `--command` thì phần sau `--` là `command`), hoặc `kubectl run webapp --image=webapp-color -- --color green` (không có `--command` thì phần sau `--` là `args`).
  
#### Tài liệu tham khảo K8s
- https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/
