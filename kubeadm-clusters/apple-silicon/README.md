# Cài đặt Kubernetes bằng kubeadm trên Apple Silicon

Cập nhật tháng 3/2024

Thư mục này chứa phần thiết lập dành riêng cho các máy Mac đời mới dùng chip Apple Silicon (bộ xử lý dòng M). Hiện VirtualBox chưa hỗ trợ tốt các máy Mac này, vì vậy chúng ta sẽ tạo các máy ảo Ubuntu bằng Multipass.

Multipass cũng chạy được trên các máy Mac đời cũ (bộ xử lý Intel Core); với các máy này bạn cũng có thể dùng [VirtualBox](../virtualbox/).

> 💡 **Bổ sung:** [Multipass](https://multipass.run/) là công cụ của Canonical giúp tạo nhanh VM Ubuntu bằng một lệnh. Trên macOS, mặc định nó dùng QEMU kết hợp Hypervisor.framework của Apple nên chạy được image Ubuntu ARM64 trên chip M-series. Lưu ý các VM khi đó là kiến trúc `arm64`, vì vậy mọi binary/image bạn tải về trong VM phải có bản `arm64` (các thành phần Kubernetes chính thức và Calico đều có).

[Bấm vào đây](./docs/01-prerequisites.md) để bắt đầu.
