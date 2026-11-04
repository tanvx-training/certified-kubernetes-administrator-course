# Kết nối

Trong bài lab này, bạn sẽ truy cập các node kubernetes bằng SSH từ `student-node`, nơi bạn đang đăng nhập.

Kiểm tra từng node. Bạn phải `exit` về `student-node` trước khi thử host tiếp theo.

```bash
ssh controlplane01
```

```bash
ssh controlplane02
```

```bash
ssh controlplane03
```

```bash
ssh loadbalancer
```

```bash
ssh node01
```

```bash
ssh node02
```

Hãy nhớ `exit` về `student-node` mỗi lần. Truy cập ssh *giữa* các node với nhau không được cấu hình. Đây là cách làm chuẩn khi thiết lập mạng: có một node mà từ đó bạn có thể đi tới tất cả các node khác.


Tiếp theo: [Thiết lập Load Balancer](./04-loadbalancer.md)<br>
Trước đó: [Tài nguyên tính toán](02-compute-resources.md)
