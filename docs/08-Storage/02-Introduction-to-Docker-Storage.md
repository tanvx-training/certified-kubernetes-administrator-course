# Giới thiệu về Docker Storage

  - Xem [Bài giảng](https://kodekloud.com/topic/introduction-to-docker-storage-3/)
  
Trong phần này, chúng ta sẽ tìm hiểu về **Docker storage**.

- Để hiểu cách lưu trữ hoạt động trong các công cụ điều phối container (container orchestration) như **Kubernetes**, trước hết cần hiểu cách lưu trữ hoạt động với container. Nắm vững cách lưu trữ hoạt động với **Docker** và các kiến thức nền tảng sẽ giúp bạn hiểu cách nó hoạt động trong **Kubernetes** dễ dàng hơn rất nhiều.

- Nếu bạn mới làm quen với **Docker**, bạn có thể học các kiến thức cơ bản về Docker qua khóa học miễn phí [Docker for the absolute beginner course](https://kodekloud.com/courses/docker-for-the-absolute-beginner/).

## Docker Storage

- Trong Docker có hai khái niệm liên quan đến lưu trữ: Storage driver và Volume driver plugin.

![class-1](../../images/class1.PNG)

#### Đầu tiên, chúng ta sẽ tìm hiểu về Storage driver.

#### Tài liệu tham khảo Docker

- https://docs.docker.com/storage/storagedriver/
- https://docs.docker.com/storage/volumes/
