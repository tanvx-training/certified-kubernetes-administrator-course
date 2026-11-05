# CKAD Exercises (Bài tập thực hành)

Bộ bài tập thực hành Kubernetes, sắp xếp theo các domain của kỳ thi [Certified Kubernetes Application Developer (CKAD)](https://www.cncf.io/certification/ckad/). Phần này được dịch sang tiếng Việt và bổ sung kiến thức từ repository [dgkanatsios/CKAD-exercises](https://github.com/dgkanatsios/CKAD-exercises) (commit `08d4d4c`) của Dimitris-Ilias Gkanatsios, theo giấy phép [MIT](LICENSE).

> 💡 **Vì sao có bài tập CKAD trong khóa CKA?** Phần lớn kỹ năng của CKAD cũng xuất hiện trong đề thi CKA: tạo Pod/Deployment/Service bằng lệnh imperative, ConfigMap/Secret, resource requests/limits, probe, Job/CronJob, NetworkPolicy, PV/PVC, Helm, CRD. Luyện các bài này giúp bạn gõ `kubectl` nhanh và chính xác hơn, điều quyết định khi thi vì thời gian rất hạn chế. Các chủ đề riêng của CKA như cài đặt và nâng cấp cluster bằng kubeadm, backup/restore etcd, troubleshooting control plane thì không có trong bộ này. Hãy ôn các phần đó ở những mục trước của repository.

## Cách sử dụng

- Ở đầu mỗi phần có breadcrumb dạng `kubernetes.io > Documentation > ...`, chỉ vị trí tài liệu liên quan trên kubernetes.io. Hãy nhớ vị trí này: khi thi bạn chỉ được mở tài liệu chính thức trong trình duyệt của máy thi.
- Nên đọc tài liệu chính thức trước khi làm bài tập của phần đó.
- Tự làm bài trước, sau đó mới bấm **Xem lời giải** để đối chiếu.
- Có thể luyện trên cluster cá nhân (kind, minikube, k3d, hoặc cluster kubeadm trong thư mục [kubeadm-clusters](../../kubeadm-clusters)) hoặc trên lab của KodeKloud / killercoda.

## Nội dung

Tỷ lệ phần trăm là trọng số theo curriculum CKAD cũ mà bộ bài tập gốc dùng để phân loại.

- [01-Core-Concepts](01-Core-Concepts.md) (13%)
- [02-Multi-Container-Pods](02-Multi-Container-Pods.md) (10%)
- [03-Pod-Design](03-Pod-Design.md) (20%)
- [04-Configuration](04-Configuration.md) (18%)
- [05-Observability](05-Observability.md) (18%)
- [06-Services-and-Networking](06-Services-and-Networking.md) (13%)
- [07-State-Persistence](07-State-Persistence.md) (8%)
- [08-Helm](08-Helm.md)
- [09-Custom-Resource-Definitions](09-Custom-Resource-Definitions.md)
- [10-Podman](10-Podman.md)
