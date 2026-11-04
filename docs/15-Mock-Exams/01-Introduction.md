# Giới thiệu về Mock Exams (Đề thi thử)

  - Đi tới [Giới thiệu về Mock Exams](https://kodekloud.com/topic/mock-exam-introduction-4/)

  > 💡 **Bổ sung – Mẹo khi làm mock exam và thi CKA thật:**
  >
  > - **Định dạng đề:** CKA là bài thi thực hành (performance-based) trên terminal, thời gian 2 giờ, khoảng 15–20 câu, điểm đạt 66%. Mỗi câu có trọng số khác nhau, được ghi ngay trong đề.
  > - **Luôn chuyển đúng môi trường:** đầu mỗi câu thường có lệnh chuyển context (`kubectl config use-context ...`) hoặc lệnh SSH sang host cần làm. Hãy copy và chạy lệnh đó **trước tiên** – làm đúng nhưng sai cluster thì vẫn mất điểm.
  > - **Alias và autocomplete:** môi trường thi thường đã có sẵn `alias k=kubectl` và bash completion. Nếu chưa có, thêm vào `~/.bashrc`: `alias k=kubectl`, `source <(kubectl completion bash)`, `complete -o default -F __start_kubectl k`. Có thể thêm `export do="--dry-run=client -o yaml"` để gõ nhanh `k run nginx --image=nginx $do > pod.yaml`.
  > - **Đừng viết YAML từ đầu:** dùng lệnh imperative kết hợp `--dry-run=client -o yaml` để sinh khung manifest, rồi sửa thêm những trường còn thiếu (volume, securityContext, toleration...). Ví dụ: `kubectl run`, `kubectl create deployment`, `kubectl create service`, `kubectl expose`, `kubectl create role/rolebinding`, `kubectl create job/cronjob`.
  > - **Tra cứu trường bằng `kubectl explain`:** ví dụ `kubectl explain pod.spec.containers.securityContext` hoặc `kubectl explain pv.spec --recursive` – nhanh hơn tìm trên web. Trong lúc thi được phép mở tài liệu chính thức tại kubernetes.io/docs (và các trang khác được liệt kê trong Candidate Handbook).
  > - **Sửa Pod đang chạy:** nhiều trường của Pod không sửa được trực tiếp. Dùng `kubectl get pod <tên> -o yaml > pod.yaml`, sửa file, rồi `kubectl replace --force -f pod.yaml` (xóa và tạo lại trong một lệnh).
  > - **Quản lý thời gian:** làm lượt đầu với các câu dễ/nhanh, câu nào quá ~5–7 phút thì đánh dấu (flag) và quay lại sau. Câu nhiều điểm nhưng khó (troubleshooting cluster, upgrade, etcd backup/restore) nên dành thời gian cuối.
  > - **Tự kiểm tra kết quả:** sau mỗi câu, chạy `kubectl get`/`kubectl describe` để xác nhận resource đã chạy (`Running`, `Bound`...) và đúng tên, namespace, label như đề yêu cầu.
