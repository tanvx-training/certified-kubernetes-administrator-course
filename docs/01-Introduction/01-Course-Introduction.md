# Giới thiệu khóa học

  - Xem [Video hướng dẫn](https://kodekloud.com/topic/course-introduction-5/)

#### Khóa học này tập trung vào phần quản trị (Administration) Kubernetes. 

## Cấu trúc khóa học

   - Bài giảng (Lectures)
   - Demo
   - Câu hỏi trắc nghiệm (Quizzes)
   - Bài tập thực hành (Practice Questions)
   - Mục hỏi đáp (Q & A)
   
## Kiến thức cần có
 
   - Docker
   - Kiến thức cơ bản về Kubernetes
     - Pod, Deployment, Service ...
   - YAML
   - Dựng một môi trường lab cơ bản với VirtualBox
   
#### Nếu bạn mới bắt đầu, chúng tôi khuyên bạn nên học khóa Kubernetes for the Absolute Beginners Hands-On trước

   - Đi tới [Khóa học Kubernetes for the Absolute Beginners Hands-On](https://kodekloud.com/courses/kubernetes-for-the-absolute-beginners-hands-on/)
   
## Mục tiêu khóa học

Mục tiêu của khóa học được xây dựng bám sát kỳ thi **`Certified Kubernetes Administration Exam (CKA)`**

- Khái niệm cốt lõi (Core Concepts)
  - Kiến trúc cluster
  - Các đối tượng API cơ bản (API Primitives)
  - Service và các thành phần mạng cơ bản khác
- Lập lịch (Scheduling)
  - Label & Selector
  - DaemonSet
  - Resource Limits
  - Nhiều scheduler (Multiple Schedulers)
  - Lập lịch thủ công (Manual Scheduling)
  - Sự kiện của scheduler (Scheduler Events)
  - Cấu hình Kubernetes Scheduler
- Logging & Monitoring
  - Giám sát các thành phần của cluster
  - Giám sát log của các thành phần cluster
  - Giám sát ứng dụng
  - Log của ứng dụng
- Quản lý vòng đời ứng dụng (Application Lifecycle Management)
  - Rolling update và rollback với Deployment
  - Cấu hình ứng dụng
  - Scale ứng dụng
  - Ứng dụng tự phục hồi (Self-Healing)
- Bảo trì cluster (Cluster Maintenance)
  - Quy trình upgrade cluster
  - Upgrade hệ điều hành
  - Các phương pháp backup và restore
 - Bảo mật (Security)
   - Xác thực & phân quyền (Authentication & Authorization)
   - Bảo mật Kubernetes
   - Network Policy
   - TLS certificate cho các thành phần cluster
   - Bảo mật image
   - Network Policy
   - Security Context
   - Lưu trữ key-value an toàn
- Lưu trữ (Storage)
  - PersistentVolume
  - Access Mode của volume
  - PersistentVolumeClaim
  - Các đối tượng lưu trữ của Kubernetes
  - Cấu hình ứng dụng dùng lưu trữ bền vững (persistent storage)
- Mạng (Networking)
  - Kiến thức nền - Mạng, Switching, Routing, công cụ
  - Kiến thức nền - Network namespace
  - Kiến thức nền - Mạng trong Docker
  - Cấu hình mạng trên các node của cluster
  - Service Networking
  - Khái niệm mạng của Pod
  - Network Load Balancer
  - Ingress
  - DNS của cluster
  - CNI
- Cài đặt, cấu hình & kiểm chứng (Installation, Configuration & Validation)
  - Thiết kế một cluster Kubernetes
  - Cài đặt master và các node
  - Bảo mật giao tiếp trong cluster
  - Cluster Kubernetes có tính sẵn sàng cao (HA)
  - Các file binary phát hành của Kubernetes
  - Cấp phát hạ tầng (Provision Infrastructure)
  - Chọn giải pháp mạng
  - Cấu hình hạ tầng Kubernetes
  - Chạy & phân tích kiểm thử end-to-end
  - Kiểm thử end-to-end cho node
- Troubleshooting
  - Lỗi ứng dụng
  - Lỗi control plane
  - Lỗi worker node
  - Mạng

> 💡 **Bổ sung:** Danh sách trên dựa theo curriculum cũ. Curriculum CKA cập nhật từ 18/02/2025 gồm 5 domain: **Troubleshooting (30%)**, **Cluster Architecture, Installation & Configuration (25%)**, **Services & Networking (20%)**, **Workloads & Scheduling (15%)** và **Storage (10%)**. Một số nội dung mới cần ôn thêm ngoài khóa học gốc: cài ứng dụng bằng **Helm** và **Kustomize**, hiểu **CRD**/operator, **Gateway API** (bên cạnh Ingress), các interface mở rộng **CNI/CSI/CRI**, dynamic provisioning với StorageClass, autoscaling workload (HPA). Ngược lại, các mục như "Run & Analyze end-to-end test" hay "Kubernetes Release Binaries" gần như không còn xuất hiện trong đề. Xem chi tiết tại [CNCF curriculum](https://github.com/cncf/curriculum).
  
## Practice Test

CKA là kỳ thi thực hành (hands-on), vì vậy việc luyện tập những gì đã học là cực kỳ quan trọng. Đó là lý do chúng tôi xây dựng một giải pháp riêng, cho phép bạn truy cập một **`Real Kubernetes Environment`** (môi trường Kubernetes thật) ngay trên trình duyệt, kèm theo **`Quiz Portal`** (cổng làm bài trắc nghiệm).
  
## Kubernetes the hard way

   - Đi tới [Kubernetes The Hard way on VirtualBox](https://github.com/mmumshad/kubernetes-the-hard-way)
