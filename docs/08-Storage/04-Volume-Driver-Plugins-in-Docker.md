# Volume Driver Plugin trong Docker

  - Xem [Bài giảng](https://kodekloud.com/topic/volume-driver-plugins-in-docker-4/)

Trong phần này, chúng ta sẽ tìm hiểu về **Volume Driver Plugin trong Docker**

- Chúng ta đã tìm hiểu về storage driver. Storage driver giúp quản lý lưu trữ trên image và container.
- Chúng ta cũng đã thấy rằng nếu muốn lưu trữ bền vững, bạn phải tạo volume. Volume không do storage driver xử lý mà do **volume driver plugin** xử lý. Volume driver plugin mặc định là `local`.
- Plugin `local` giúp tạo volume trên Docker host và lưu dữ liệu của nó trong thư mục `/var/lib/docker/volumes/`.
- Có nhiều volume driver plugin khác cho phép tạo volume trên các giải pháp của bên thứ ba như Azure File Storage, DigitalOcean Block Storage, Portworx, Google Compute Persistent Disk, v.v.


![class-9](../../images/class9.PNG)


- Khi chạy Docker container, bạn có thể chọn một volume driver cụ thể, ví dụ RexRay EBS để cấp phát (provision) volume từ Amazon EBS. Lệnh dưới đây sẽ tạo một container và gắn một volume từ AWS cloud. Khi container dừng, dữ liệu của bạn vẫn an toàn trên cloud.

```
$ docker run -it --name mysql --volume-driver rexray/ebs --mount src=ebs-vol,target=/var/lib/mysql mysql
```


![class-10](../../images/class10.PNG)

> 💡 **Bổ sung:** Dự án RexRay hiện đã ngừng phát triển (archived). Trong Kubernetes, bạn không dùng Docker volume plugin mà dùng các **CSI driver** (ví dụ AWS EBS CSI driver `ebs.csi.aws.com`) — xem phần Container Storage Interface tiếp theo.






#### Tài liệu tham khảo Docker

- https://docs.docker.com/engine/extend/legacy_plugins/
- https://github.com/rexray/rexray

