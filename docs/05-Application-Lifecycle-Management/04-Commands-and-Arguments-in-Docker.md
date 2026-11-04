# Commands và Arguments trong Docker
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/commands-and-arguments-in-docker/)
  
Trong phần này, chúng ta sẽ tìm hiểu về command và argument trong Docker.

- Chạy một Docker container
  ```
  $ docker run ubuntu
  ```
- Liệt kê các container đang chạy
  ```
  $ docker ps 
  ```
- Liệt kê tất cả container, kể cả các container đã dừng
  ```
  $ docker ps -a
  ```
  
  ![dc](../../images/dc.PNG)

Khi chạy `docker run ubuntu`, container khởi động rồi thoát ngay lập tức. Lý do là image ubuntu có lệnh mặc định (CMD) là `bash`; bash là một shell chờ input từ terminal, nhưng mặc định Docker không gắn terminal vào container, nên bash không tìm thấy terminal và thoát, kéo theo container cũng dừng.
  
#### Khác với máy ảo (VM), container không được thiết kế để chạy cả một hệ điều hành.
- Container được dùng để chạy một tác vụ hoặc tiến trình cụ thể, ví dụ chạy một instance của web server, application server hay database server... Container chỉ tồn tại chừng nào tiến trình bên trong còn chạy; khi tiến trình kết thúc hoặc bị crash, container sẽ thoát.

  ![ex](../../images/ex.PNG)

Tiến trình chạy bên trong container được định nghĩa bởi chỉ thị `CMD` trong Dockerfile, ví dụ image nginx dùng `CMD ["nginx"]`, image mysql dùng `CMD ["mysqld"]`.
  
#### Làm sao để chỉ định một command khác khi khởi động container?
- Một cách là thêm command vào cuối lệnh docker run; khi đó command này sẽ ghi đè (override) command mặc định được định nghĩa trong image.
  ```
  $ docker run ubuntu sleep 5
  ```
- Như vậy khi container khởi động, nó sẽ chạy chương trình sleep, đợi 5 giây rồi thoát. Làm thế nào để thay đổi này trở thành mặc định (lâu dài)?
  
  ![sleep](../../images/sleep.PNG)

  Ta tạo một image mới dựa trên image ubuntu và chỉ định command mới trong Dockerfile.
  
- Có nhiều cách để chỉ định command: viết command ở dạng shell (shell form, vd `CMD sleep 5`) hoặc dạng mảng JSON (exec form, vd `CMD ["sleep", "5"]`). Với dạng JSON, phần tử đầu tiên phải là file thực thi, và command với tham số phải là các phần tử riêng biệt (không viết `["sleep 5"]`).
 
  ![sleep1](../../images/sleep1.PNG)
  
- Build Docker image
  ```
  $ docker build -t ubuntu-sleeper .
  ```
- Chạy Docker container
  ```
  $ docker run ubuntu-sleeper
  ```
  
  ![sleep2](../../images/sleep2.PNG)
  
## Chỉ thị Entrypoint
- Chỉ thị `ENTRYPOINT` giống chỉ thị `CMD` ở chỗ nó chỉ định chương trình sẽ được chạy khi container khởi động. Điểm khác biệt: những gì bạn truyền thêm trên dòng lệnh sẽ được **nối thêm (append)** vào sau ENTRYPOINT, thay vì thay thế hoàn toàn như với CMD.

> 💡 **Bổ sung:** Tóm tắt cách CMD và ENTRYPOINT phối hợp:
> - Chỉ có `CMD ["sleep", "5"]`: `docker run ubuntu-sleeper 10` → tham số `10` **thay thế toàn bộ** CMD, Docker cố chạy lệnh `10` và báo lỗi.
> - Chỉ có `ENTRYPOINT ["sleep"]`: `docker run ubuntu-sleeper 10` → chạy `sleep 10`. Nhưng nếu không truyền tham số (`docker run ubuntu-sleeper`) thì chạy `sleep` không có tham số và báo lỗi thiếu operand.
> - Kết hợp `ENTRYPOINT ["sleep"]` và `CMD ["5"]`: không truyền gì → chạy `sleep 5` (CMD đóng vai trò tham số mặc định); truyền `10` → chạy `sleep 10`. Để kết hợp như vậy, cả hai phải viết ở dạng JSON (exec form).
> - Muốn ghi đè ENTRYPOINT khi chạy: `docker run --entrypoint sleep2.0 ubuntu-sleeper 10`.

#### Tài liệu tham khảo
- https://docs.docker.com/engine/reference/builder/#cmd
