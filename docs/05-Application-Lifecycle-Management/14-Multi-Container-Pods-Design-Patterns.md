# Các Design Pattern cho Multi-Container Pod
  - Đi tới [trang Design Pattern](https://kodekloud.com/topic/multi-container-pods-design-patterns/)
  
  ![dp](../../images/dp.PNG)

Có 3 design pattern phổ biến cho multi-container Pod:
- **Sidecar**: container phụ hỗ trợ container chính, ví dụ một logging agent thu thập log của web server và gửi về hệ thống log tập trung.
- **Adapter**: chuẩn hóa/chuyển đổi output của container chính trước khi gửi đi, ví dụ chuyển các định dạng log khác nhau của nhiều ứng dụng về cùng một định dạng chung.
- **Ambassador**: làm proxy cho kết nối từ container chính ra bên ngoài, ví dụ ứng dụng chỉ cần kết nối tới `localhost`, còn ambassador quyết định chuyển tiếp tới database dev, test hay prod.

> 💡 **Bổ sung:** Từ Kubernetes v1.29 (beta, bật mặc định; GA ở v1.33), sidecar có thể được khai báo dưới dạng **native sidecar**: đặt trong `initContainers` với `restartPolicy: Always`. Native sidecar khởi động trước container chính, chạy suốt vòng đời Pod và được dừng sau container chính — giải quyết vấn đề sidecar không kết thúc làm Job không bao giờ hoàn thành. Chi tiết xem bài [Init Containers](15.Init-Containers.md).
  
#### Tài liệu tham khảo K8s
- https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/
- https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/
