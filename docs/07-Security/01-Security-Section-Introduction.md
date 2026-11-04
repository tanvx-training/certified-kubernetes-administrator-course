# Giới thiệu phần Security (Bảo mật)
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/security-section-introduction/)

Trong phần này, chúng ta sẽ có cái nhìn tổng quan về các chủ đề bảo mật sẽ học:
- Các cơ chế bảo mật cơ bản của Kubernetes (Security Primitives)
- Authentication (xác thực)
- TLS certificate cho các thành phần của cluster
- Bảo mật kho lưu trữ key-value (etcd)
- Authorization (phân quyền)
- Bảo mật image
- Security Context
- Network Policy

> 💡 **Bổ sung:** Theo curriculum CKA hiện tại, mảng bảo mật nằm chủ yếu trong domain *Cluster Architecture, Installation & Configuration* (quản lý RBAC, quản lý certificate/kubeconfig) và *Services & Networking* (Network Policy). Các kỹ năng hay gặp khi thi: tạo Role/ClusterRole/RoleBinding, tạo user bằng CertificateSigningRequest, chuyển context trong kubeconfig, và viết NetworkPolicy.
