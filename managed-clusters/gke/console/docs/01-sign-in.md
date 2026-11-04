# Khởi tạo lab

Cập nhật lần cuối: tháng 3 năm 2025

Trong hướng dẫn này, chúng ta sẽ triển khai một GKE cluster tuân thủ các chính sách bảo mật và sử dụng của KodeKloud playground. Lưu ý rằng bạn phải làm theo hướng dẫn này *chính xác*, nếu không có thể gặp lỗi về quyền (permission).

Vì đây là sản phẩm của Google nên nó sẽ hoạt động tốt nhất trên trình duyệt Chrome. Các trình duyệt phổ biến khác *hẳn là* cũng dùng được.

### QUAN TRỌNG

Lưu ý rằng nếu trình duyệt của bạn đã đăng nhập sẵn một tài khoản Google, nó có thể cố dùng tài khoản đó và bạn sẽ gặp lỗi và/hoặc bị đăng xuất khỏi tài khoản Google của mình. Nếu bạn đã có sẵn một tài khoản GCP cá nhân và đang đăng nhập Google, thì nó *sẽ tự động dùng tài khoản này* và không kết nối bạn tới playground! Nếu rơi vào một trong hai trường hợp trên, hãy dùng cửa sổ ẩn danh (incognito) để đăng nhập vào tài khoản playground.

### Khởi tạo GCP playground lab

Điều kiện tiên quyết: có gói đăng ký KodeKloud Premium

Truy cập link sau: https://kodekloud.com/playgrounds/playground-google-cloud và bấm `START LAB` để yêu cầu một GCP Playground instance mới. Sau vài giây, bạn sẽ nhận được một tài khoản GCP portal tạm thời như sau:

![image](../images/01-sign-in.png)

Sao chép **Console Link** bằng cách bấm nút copy ở bên phải của nó, dán vào trình duyệt và đăng nhập bằng thông tin đăng nhập được cung cấp. Nhập username được cấp cho lab vào ô `email or phone` trong hộp thoại đăng nhập.

Trong hộp thoại tiếp theo, chọn quốc gia của bạn và đồng ý với điều khoản dịch vụ (terms of service). Để trống (không tích) ô **Email updates**.

Tiếp theo: [Khởi tạo Cluster](./02-create-cluster.md)



