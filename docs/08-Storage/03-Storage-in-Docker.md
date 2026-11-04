# Storage trong Docker

  - Xem [Bài giảng](https://kodekloud.com/topic/storage-in-docker-2/)

Trong phần này, chúng ta sẽ tìm hiểu về **Storage driver và Filesystem** (hệ thống file) của Docker.

  - Chúng ta sẽ xem Docker lưu dữ liệu ở đâu, như thế nào và cách nó quản lý filesystem của các container.

## Filesystem

- Hãy xem cách Docker lưu dữ liệu trên filesystem cục bộ. Khi bạn cài đặt Docker lần đầu trên một hệ thống, nó sẽ tạo cấu trúc thư mục tại `/var/lib/docker`.

  ```
  $ cd /var/lib/docker/

  ```

![class-2](../../images/class2.PNG)

- Bên trong có nhiều thư mục con như `aufs`, `containers`, `image`, `volumes`, v.v.
- Đây là nơi Docker lưu toàn bộ dữ liệu của nó theo mặc định.
- Mọi file liên quan đến container được lưu trong thư mục `containers`, các file liên quan đến image được lưu trong thư mục `image`. Mọi volume do Docker container tạo ra đều nằm trong thư mục `volumes`.
- Hiện tại, chúng ta chỉ cần hiểu Docker lưu các file của image và container ở đâu và theo định dạng nào.
- Để hiểu điều đó, ta cần hiểu kiến trúc phân lớp (layered architecture) của Docker.

> 💡 **Bổ sung:** Trên các bản Docker hiện đại, storage driver mặc định là `overlay2` nên bạn sẽ thấy thư mục `overlay2` thay vì `aufs`. Lưu ý thêm: Kubernetes đã gỡ bỏ dockershim từ v1.24, các cluster hiện nay (kể cả môi trường thi CKA) thường dùng **containerd** (hoặc CRI-O) làm container runtime; containerd lưu dữ liệu ở `/var/lib/containerd`. Tuy nhiên các khái niệm về layer, copy-on-write và volume trình bày dưới đây vẫn hoàn toàn đúng.

## Kiến trúc phân lớp (Layered Architecture)

- Khi Docker build image, nó build theo kiến trúc phân lớp. Mỗi dòng lệnh (instruction) trong Dockerfile tạo ra một layer mới trong Docker image, chỉ chứa những thay đổi so với layer trước đó.
- Như bạn thấy trong hình, layer 1 là layer nền Ubuntu, layer 2 cài đặt các package, layer 3 cài các package Python, layer 4 cập nhật mã nguồn, layer 5 cập nhật entry point của image. Vì mỗi layer chỉ lưu những thay đổi so với layer trước, điều này cũng được phản ánh qua kích thước của từng layer.

![class-3](../../images/class3.PNG)

- Để hiểu rõ hơn lợi ích của kiến trúc phân lớp, hãy xét một Dockerfile khác, rất giống với ứng dụng đầu tiên của chúng ta, chỉ khác mã nguồn và entry point.
- Khi build image này, Docker sẽ không build lại ba layer đầu tiên mà tái sử dụng chính ba layer đã build cho ứng dụng đầu tiên từ cache. Docker chỉ tạo hai layer cuối với mã nguồn mới và entry point mới.
- Nhờ vậy, Docker build image nhanh hơn và tiết kiệm dung lượng đĩa hiệu quả. Điều này cũng áp dụng khi bạn cập nhật mã nguồn ứng dụng: Docker đơn giản là tái sử dụng tất cả các layer trước đó từ cache và nhanh chóng build lại image ứng dụng bằng cách cập nhật mã nguồn mới nhất.

![class-4](../../images/class4.PNG)

- Hãy sắp xếp lại các layer theo thứ tự từ dưới lên để dễ hiểu hơn. Tất cả các layer này được tạo ra khi chạy lệnh `docker build` để hình thành Docker image cuối cùng. Khi build xong, bạn không thể sửa nội dung của các layer này; chúng là **read-only** (chỉ đọc) và bạn chỉ có thể thay đổi chúng bằng cách thực hiện một lần build mới.

![class-5](../../images/class5.PNG)

- Khi bạn chạy một container từ image này bằng lệnh **docker run**, Docker tạo container dựa trên các layer đó và tạo thêm một **writable layer** (layer có thể ghi) mới nằm trên cùng các image layer. Writable layer được dùng để lưu dữ liệu do container tạo ra, chẳng hạn file log do ứng dụng ghi, các file tạm do container sinh ra.

![class-6](../../images/class6.PNG)

- Khi container bị xóa, layer này cùng toàn bộ thay đổi lưu trong nó cũng bị xóa theo. Hãy nhớ rằng cùng một image layer được chia sẻ bởi tất cả các container tạo từ image đó.
- Ví dụ, nếu tôi tạo một file mới tên `temp.txt` trong container vừa tạo, file này nằm trong layer đọc-ghi (read-write).
- Các file trong image layer là read-only, nghĩa là bạn không thể chỉnh sửa gì trong các layer đó.


![class-7](../../images/class7.PNG)

- Hãy lấy ví dụ với mã nguồn ứng dụng. Vì mã nguồn được "đóng gói" sẵn vào image, nó là một phần của image và do đó là read-only. Vậy sau khi chạy container, nếu tôi muốn sửa mã nguồn thì sao?
- Tôi vẫn có thể sửa file này, nhưng trước khi lưu file đã sửa, Docker sẽ tự động tạo một bản sao của file đó trong layer read-write, và tôi sẽ sửa trên bản sao này. Mọi chỉnh sửa về sau đều thực hiện trên bản sao trong layer read-write. Cơ chế này gọi là **copy-on-write** (sao chép khi ghi).
- Việc image layer là read-only chỉ có nghĩa là các file trong các layer này sẽ không bị sửa đổi trong chính image. Vì vậy, image luôn giữ nguyên cho đến khi bạn build lại bằng lệnh `docker build`. Nếu container bị xóa, toàn bộ dữ liệu lưu trong container layer cũng bị xóa theo.

## Volumes

- Để có dữ liệu bền vững (persistent) cho container, ta cần tạo một volume bằng lệnh `docker volume create`. Khi chạy lệnh `docker volume create data_volume`, Docker tạo một thư mục tên `data_volume` trong thư mục `/var/lib/docker/volumes`.

  ```
  $ docker volume create data_volume

  $ ls -l /var/lib/docker/volumes/

  drwxr-xr-x 3 root root  4096 Aug 01 17:53 data_volume

  $ docker volume ls 

  DRIVER              VOLUME NAME
	local               data_volume
  ```
- Khi chạy Docker container bằng lệnh `docker run`, tôi có thể mount volume này vào bên trong container bằng tùy chọn `-v`.
- Cụ thể, khi chạy `docker run -v`, tôi chỉ định tên volume vừa tạo, theo sau là dấu hai chấm (:) và vị trí bên trong container — ở đây là vị trí mặc định MySQL lưu dữ liệu, tức `/var/lib/mysql` — và cuối cùng là tên image MySQL. Lệnh này tạo một container mới và mount data volume vào đó.

	```
	$ docker run -v data_volume:/var/lib/mysql mysql
  ```
- Ngay cả khi container bị xóa, dữ liệu vẫn còn nguyên.
- Nếu bạn chưa tạo Docker volume trước khi chạy `docker run` thì sao? Trong trường hợp này, Docker sẽ tự động tạo một volume tên `data_volume2` và mount nó vào container.

  ```
  $ docker run -v data_volume2:/var/lib/mysql mysql

  $ docker volume ls

	DRIVER              VOLUME NAME
	local               data_volume
	local               data_volume2
  ```
- Bạn sẽ thấy tất cả các volume này nếu liệt kê nội dung thư mục `/var/lib/docker/volumes`. Cách này gọi là **Volume mounting**.
- Vậy nếu dữ liệu của chúng ta đã nằm sẵn ở một vị trí khác thì sao?
- Giả sử ta có một ổ lưu trữ ngoài trên Docker host tại đường dẫn `/data` và muốn lưu dữ liệu database trên đó thay vì trong thư mục mặc định `/var/lib/docker/volumes`. Khi đó, ta vẫn chạy container bằng lệnh `docker run -v`, nhưng cung cấp đường dẫn đầy đủ tới thư mục muốn mount, tức `/data/mysql`. Docker sẽ tạo container và mount thư mục đó vào container. Cách này gọi là **Bind mounting**.
- Như vậy có hai kiểu mount: volume mount và bind mount.
  * Volume mount: mount một volume từ thư mục volumes của Docker; bind mount: mount một thư mục từ bất kỳ vị trí nào trên Docker host.
- Thay vì tùy chọn `-v`, nên ưu tiên dùng tùy chọn `--mount` (cú pháp mới, tường minh hơn).

  ```
  $ mkdir -p /data/mysql

  $ docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
  ```
- Vậy ai chịu trách nhiệm thực hiện tất cả các thao tác này?
- Duy trì kiến trúc phân lớp, tạo writable layer, di chuyển file giữa các layer để thực hiện copy-on-write, v.v. Đó chính là **Storage Driver**.
- Docker sử dụng storage driver để hiện thực kiến trúc phân lớp.

![class-8](../../images/class8.PNG)

#### Các Storage Driver phổ biến

- AUFS
- ZFS
- BTRFS
- Device Mapper
- Overlay
- Overlay2

- Việc lựa chọn storage driver phụ thuộc vào hệ điều hành bên dưới. Docker sẽ tự động chọn storage driver tốt nhất hiện có dựa trên hệ điều hành.

> 💡 **Bổ sung:** Hiện nay `overlay2` là storage driver mặc định và được khuyến nghị trên hầu hết các bản phân phối Linux; `aufs` và `devicemapper` đã bị deprecated/gỡ bỏ trong các phiên bản Docker Engine mới. Có thể kiểm tra driver đang dùng bằng `docker info` (dòng `Storage Driver`).

#### Tài liệu tham khảo Docker

- https://docs.docker.com/storage/
- https://docs.docker.com/engine/reference/commandline/volume_create/
- https://docs.docker.com/engine/reference/commandline/volume_ls/
