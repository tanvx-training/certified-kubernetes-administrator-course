# Điều kiện tiên quyết

* Có quyền truy cập vào một tài khoản AWS. Có thể là [KodeKloud Playground](https://kodekloud.com/topic/playground-aws/), hoặc tài khoản của riêng bạn.

Chúng tôi đã đặt sẵn biến môi trường `PRIMARY_IP` trên tất cả các VM, đây là địa chỉ IP mà các thành phần của kube nên sử dụng. `PRIMARY_IP` được định nghĩa là địa chỉ IP của network interface trên node nối với mạng có default gateway, và cũng là interface mà node dùng để nói chuyện với các node khác. Nếu bạn quan tâm, biến này được gán bằng kết quả của lệnh sau

```bash
ip route | grep default | awk '{ print $9 }'
```


Tiếp theo: [Tài nguyên tính toán](02-compute-resources.md)
