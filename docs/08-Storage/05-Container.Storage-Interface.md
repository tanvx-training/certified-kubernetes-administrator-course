# Container Storage Interface

  - Xem [Bài giảng](https://kodekloud.com/topic/container-storage-interface/)

Trong phần này, chúng ta sẽ tìm hiểu về **Container Storage Interface (CSI)**

## Container Runtime Interface

- Trước đây, Kubernetes chỉ dùng Docker làm container runtime, và toàn bộ code để làm việc với Docker được nhúng thẳng vào mã nguồn Kubernetes. Sau đó xuất hiện các container runtime khác như rkt và CRI-O.
- Container Runtime Interface (CRI) là một chuẩn định nghĩa cách một giải pháp điều phối như Kubernetes giao tiếp với các container runtime như Docker. Nếu có container runtime mới được phát triển, chúng chỉ cần tuân theo chuẩn CRI.

![class-11](../../images/class11.PNG)

> 💡 **Bổ sung:** Docker Engine không hỗ trợ CRI trực tiếp nên Kubernetes phải dùng lớp trung gian dockershim. Dockershim đã bị gỡ khỏi Kubernetes từ **v1.24**; hiện nay các runtime phổ biến là **containerd** và **CRI-O** (muốn dùng Docker Engine phải cài thêm `cri-dockerd`). Dự án rkt cũng đã ngừng phát triển.


## Container Networking Interface

- Để hỗ trợ nhiều giải pháp mạng khác nhau, Container Networking Interface (CNI) được giới thiệu. Bất kỳ nhà cung cấp giải pháp mạng mới nào cũng chỉ cần phát triển plugin dựa trên chuẩn CNI để giải pháp của họ hoạt động được với Kubernetes.

![class-12](../../images/class12.PNG)



## Container Storage Interface

- Container Storage Interface (CSI) được phát triển để hỗ trợ nhiều giải pháp lưu trữ khác nhau. Với CSI, bạn có thể tự viết driver cho hệ thống lưu trữ của mình để nó hoạt động với Kubernetes. Ví dụ: Portworx, Amazon EBS, Azure Disk, GlusterFS, v.v.
- CSI không phải là chuẩn riêng của Kubernetes. Nó được thiết kế như một chuẩn chung, và nếu được hiện thực, cho phép bất kỳ công cụ điều phối container nào làm việc với bất kỳ nhà cung cấp lưu trữ nào có plugin hỗ trợ. Kubernetes, Cloud Foundry và Mesos đều đã hỗ trợ CSI.
- CSI định nghĩa một tập các RPC (remote procedure call — lời gọi thủ tục từ xa) mà container orchestrator sẽ gọi. Các storage driver phải hiện thực những RPC này.

![class-13](../../images/class13.PNG)

> 💡 **Bổ sung:** Ví dụ về các RPC trong CSI: khi một Pod cần volume, Kubernetes gọi `CreateVolume` để storage driver cấp phát volume mới trên hệ thống lưu trữ; khi xóa volume thì gọi `DeleteVolume`; ngoài ra còn có `ControllerPublishVolume` (gắn volume vào node), `NodePublishVolume` (mount volume vào Pod)... Trong Kubernetes hiện đại, các volume plugin "in-tree" (nằm sẵn trong mã nguồn K8s như `kubernetes.io/aws-ebs`, `kubernetes.io/gce-pd`, `kubernetes.io/azure-disk`) đã được **migrate sang CSI** và phần lớn đã bị gỡ khỏi mã nguồn; bạn nên dùng CSI driver tương ứng (vd `ebs.csi.aws.com`, `pd.csi.storage.gke.io`, `disk.csi.azure.com`). Xem danh sách CSI driver trong cluster bằng `kubectl get csidrivers`.



#### Tài liệu tham khảo Container Storage Interface

- https://github.com/container-storage-interface/spec
- https://kubernetes-csi.github.io/docs/
- http://mesos.apache.org/documentation/latest/csi/
- https://www.nomadproject.io/docs/internals/plugins/csi#volume-lifecycle
