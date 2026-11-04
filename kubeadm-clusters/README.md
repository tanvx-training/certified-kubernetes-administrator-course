# Cài đặt cluster bằng Kubeadm

Trong phần này, chúng tôi giới thiệu nhiều bài lab khác nhau để tự dựng cluster bằng kubeadm.

* AWS - Sử dụng KodeKloud AWS Playground

    Trong các bài lab này, terraform được dùng để tạo sẵn các máy ảo, phần cấu hình cluster sẽ do bạn tự thực hiện.
    * [Cluster 3 node](./aws/)
    * [Cluster 5 node có tính sẵn sàng cao (Highly Available)](./aws-ha/)

* Apple Silicon

    Dành cho người dùng máy Mac chip M-series: bài lab này tạo một cluster 2 hoặc 3 node (tùy theo dung lượng RAM của máy Mac) bằng [Multipass](https://multipass.run/).

    * [Cluster 2/3 node](./apple-silicon/)

* Vagrant và VirtualBox

    Dành cho người dùng Windows và Mac x86: tạo một cluster 3 node ngay trên laptop của bạn.

    * [Cluster 3 node](./virtualbox/)

> 💡 **Bổ sung:** Đề thi CKA có thể yêu cầu cài đặt hoặc upgrade cluster bằng `kubeadm`, vì vậy tự tay dựng cluster ít nhất một lần là cách ôn tập rất hiệu quả. Hãy chú ý các bước chuẩn bị node (kernel module, sysctl, cấu hình cgroup của containerd), `kubeadm init`, cài CNI và `kubeadm join` — đây là những chỗ dễ sai nhất.
