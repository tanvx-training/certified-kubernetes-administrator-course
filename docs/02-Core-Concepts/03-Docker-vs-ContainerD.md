# Docker và ContainerD

Trong phần này, chúng ta sẽ tìm hiểu sự khác nhau giữa Docker và ContainerD


Bạn sẽ bắt gặp Docker và `containerd` rất nhiều lần. Khi đọc các blog hoặc tài liệu cũ, bạn sẽ thấy Docker được nhắc đến cùng với Kubernetes, còn khi đọc các bài viết mới hơn thì bạn sẽ thấy `containerd`, và bạn sẽ tự hỏi hai thứ này khác nhau ra sao. Ngoài ra còn có một vài công cụ CLI như `ctr`, `crictl` hay `nerdctl`, và bạn sẽ thắc mắc chúng là gì, nên dùng công cụ nào. Đó chính là những điều mình sẽ giải thích trong phần này.

![](../../images/02-03-01.png)

Hãy quay ngược thời gian về lúc khởi đầu của kỷ nguyên container. Ban đầu chỉ có Docker. Cũng có vài công cụ khác như Rocket ([rkt](https://www.redhat.com/en/topics/containers/what-is-rkt)), nhưng trải nghiệm người dùng của Docker khiến việc làm việc với container trở nên cực kỳ đơn giản, nên Docker trở thành công cụ container thống trị. Sau đó Kubernetes ra đời để điều phối (orchestrate) Docker — ban đầu Kubernetes được xây dựng riêng để điều phối Docker, nên hai bên gắn chặt với nhau, và thời đó Kubernetes chỉ làm việc với Docker, không hỗ trợ giải pháp container nào khác.

![](../../images/02-03-02.png)

Rồi Kubernetes ngày càng phổ biến với vai trò container orchestrator, và các container runtime khác như `rkt` cũng muốn tham gia. Người dùng Kubernetes cần nó làm việc được với cả những container runtime khác ngoài Docker, vì vậy Kubernetes giới thiệu một interface gọi là Container Runtime Interface, hay CRI. CRI cho phép bất kỳ nhà cung cấp nào cũng có thể làm container runtime cho Kubernetes, miễn là tuân thủ chuẩn OCI.
OCI là viết tắt của Open Container Initiative, gồm image spec và runtime spec. Image spec là đặc tả về cách một image phải được build. Runtime spec định nghĩa các tiêu chuẩn về cách một container runtime phải được phát triển. Dựa trên các tiêu chuẩn này, bất kỳ ai cũng có thể xây dựng một container runtime để mọi người dùng với Kubernetes — đó là ý tưởng chính.

![](../../images/02-03-03.png)

Nhờ vậy, `rkt` và các container runtime khác tuân thủ chuẩn OCI được Kubernetes hỗ trợ làm container runtime thông qua CRI. Tuy nhiên, Docker không được xây dựng để hỗ trợ chuẩn CRI — nhớ rằng Docker ra đời từ rất lâu trước khi có CRI — trong khi Docker vẫn là công cụ container được dùng nhiều nhất. Vì thế Kubernetes vẫn phải tiếp tục hỗ trợ Docker, và họ giới thiệu thứ gọi là `dockershim`: một cách "chắp vá" nhưng tạm thời để hỗ trợ Docker nằm ngoài CRI.

![](../../images/02-03-04.png)

Trong khi hầu hết các container runtime khác làm việc qua CRI, Docker vẫn tiếp tục hoạt động mà không cần nó. Bạn thấy đấy, Docker không chỉ đơn thuần là một container runtime. Docker là tập hợp của nhiều công cụ: Docker CLI, Docker API, các công cụ build giúp tạo image, hỗ trợ volume, bảo mật, và cuối cùng là container runtime tên là `runc` cùng với daemon quản lý `runc`, gọi là `containerd`. containerd tương thích CRI và có thể làm việc trực tiếp với Kubernetes như mọi runtime khác, nên containerd có thể được dùng làm runtime riêng, tách biệt khỏi Docker.

![](../../images/02-03-05.png)

Vậy là giờ chúng ta có containerd là một runtime riêng, và Docker riêng. Kubernetes vẫn tiếp tục duy trì hỗ trợ trực tiếp Docker Engine, nhưng việc phải duy trì dockershim là công sức không cần thiết và gây thêm phức tạp, nên ở bản phát hành v1.24, Kubernetes quyết định gỡ bỏ hoàn toàn dockershim, đồng nghĩa với việc ngừng hỗ trợ Docker. Tuy nhiên, tất cả các image đã được build bằng Docker vẫn tiếp tục hoạt động, vì Docker tuân theo image spec của chuẩn OCI. Mọi image do Docker build đều tuân thủ chuẩn đó nên vẫn chạy được với containerd; chỉ có bản thân Docker là bị loại khỏi danh sách runtime được Kubernetes hỗ trợ. Đó là toàn bộ câu chuyện; giờ hãy đi sâu hơn vào containerd.

> 💡 **Bổ sung:** "Kubernetes bỏ Docker" chỉ có nghĩa là kubelet không còn nói chuyện trực tiếp với Docker Engine qua dockershim. Bạn vẫn dùng Docker để build image bình thường. Nếu vẫn muốn dùng Docker Engine làm runtime cho Kubernetes, có thể cài adapter [`cri-dockerd`](https://github.com/Mirantis/cri-dockerd) (do Mirantis duy trì). Ngoài containerd, runtime CRI phổ biến khác là **CRI-O**. Dự án `rkt` đã ngừng phát triển và được lưu trữ (archived) từ năm 2020.

![](../../images/02-03-06.png)

Mặc dù containerd là một phần của Docker, nó hiện là một dự án độc lập, là thành viên của [CNCF](https://www.cncf.io/) với trạng thái [graduated](https://www.cncf.io/projects/). Vì vậy bạn có thể cài containerd riêng mà không cần cài Docker; nếu không thực sự cần các tính năng khác của Docker, bạn chỉ cần cài containerd. Trước đây khi có Docker, chúng ta thường chạy container bằng lệnh `docker run`; vậy nếu không cài Docker thì làm sao chạy container chỉ với containerd? Khi cài containerd, nó đi kèm một công cụ dòng lệnh tên là [ctr](https://github.com/projectatomic/containerd/blob/master/docs/cli.md#client-cli). Công cụ này được làm ra chỉ để debug containerd, không thân thiện với người dùng vì chỉ hỗ trợ một tập tính năng hạn chế — và đó là tất cả những gì bạn thấy trong trang tài liệu của nó. Với những gì nằm ngoài tập tính năng hạn chế đó, bạn phải gọi API trực tiếp tới containerd, đây không phải cách thuận tiện nhất để vận hành.

Để bạn hình dung, lệnh `ctr` có thể dùng cho các thao tác cơ bản liên quan đến container như pull image; ví dụ để pull image redis, bạn chạy

```
ctr images pull docker.io/library/redis:alpine
```

Để chạy một container, ta dùng lệnh `ctr run`

```
ctr run docker.io/library/redis:alpine redis
```

Nhưng như đã nói, công cụ này chỉ dùng để debug containerd, không thân thiện với người dùng và không nên dùng để quản lý container trong môi trường production.
Một lựa chọn tốt hơn được khuyến nghị là công cụ [nerdctl](https://github.com/containerd/nerdctl#nerdctl-docker-compatible-cli-for-container). `nerdctl` là một công cụ dòng lệnh rất giống Docker — một CLI kiểu Docker dành cho containerd. Nó hỗ trợ hầu hết các tùy chọn CLI mà Docker hỗ trợ, và ngoài ra còn có lợi thế là cho phép truy cập các tính năng mới nhất được hiện thực trong containerd, ví dụ làm việc với container image được mã hóa, hay những tính năng mới khác mà sau này mới được đưa vào lệnh Docker thông thường. Nó cũng hỗ trợ lazy pulling image, phân phối image dạng P2P, ký và xác minh image, và namespace trong Kubernetes — những thứ Docker không có. `nerdctl` hoạt động rất giống Docker CLI: thay vì `docker`, bạn chỉ cần thay bằng `nerdctl`, và nó có thể chạy gần như mọi lệnh Docker tương tác với container, như sau

![](../../images/02-03-07.png)

Khá dễ và đơn giản. Sau khi đã nói về `ctr` và `nerdctl`, điều quan trọng là tìm hiểu thêm một công cụ dòng lệnh khác tên là [crictl](https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md#container-runtime-interface-cri-cli). Trước đó chúng ta đã nói về CRI — một interface duy nhất dùng để kết nối các container runtime tương thích CRI như containerd, `rkt` và các runtime khác. `crictl` là tiện ích dòng lệnh dùng để tương tác với container runtime tương thích CRI, tức là tương tác từ góc nhìn của Kubernetes. Công cụ này do cộng đồng Kubernetes phát triển và duy trì, và nó hoạt động với mọi container runtime khác nhau. Trong khi `ctr` và `nerdctl` do cộng đồng containerd xây dựng riêng cho containerd, thì `crictl` được làm từ góc nhìn Kubernetes và hoạt động với nhiều container runtime khác nhau.

`crictl` phải được cài riêng và được dùng để kiểm tra (inspect) và debug container runtime. Khác với Docker hay `nerdctl`, nó không được dùng để tạo container, mà chủ yếu là một công cụ debug. Về mặt kỹ thuật bạn có thể tạo container bằng `crictl` nhưng không dễ, và chỉ nên làm vậy cho một số mục đích debug đặc biệt. Hãy nhớ rằng nó hoạt động cùng với kubelet. Ta biết kubelet chịu trách nhiệm đảm bảo một số lượng container hoặc Pod nhất định luôn có trên node tại mọi thời điểm. Nếu bạn dùng `crictl` để tạo container, cuối cùng kubelet sẽ xóa chúng đi, vì kubelet không biết đến những container hay Pod được tạo ra ngoài tầm kiểm soát của nó — thấy cái nào lạ là nó xóa. Vì những lý do đó, hãy nhớ rằng `crictl` chỉ dùng cho mục đích debug, truy cập vào container, v.v.

Hãy xem một vài ví dụ dòng lệnh. Bạn chỉ cần chạy lệnh `crictl`; nó có thể dùng cho các thao tác cơ bản liên quan đến container như pull image, liệt kê image hiện có, liệt kê container — rất giống lệnh Docker. Trong Docker bạn chạy lệnh `ps`, còn ở đây bạn chạy `crictl ps`. Để chạy một lệnh bên trong container, với Docker ta dùng lệnh `exec`, ở đây cũng vậy, với các tùy chọn tương tự như `-i` và `-t`, kèm theo container id. Để xem log, bạn dùng lệnh `crictl logs`, cũng rất giống lệnh Docker.

![](../../images/02-03-08.png)

Một khác biệt lớn là lệnh `crictl` còn nhận biết được Pod, nên bạn có thể liệt kê Pod bằng lệnh `crictl pods` — điều mà Docker không hề biết đến. Trước đây khi làm việc với Kubernetes, chúng ta dùng lệnh Docker rất nhiều để troubleshoot container và xem log, đặc biệt trên các worker node; giờ bạn sẽ dùng lệnh `crictl` để làm việc đó. Cú pháp rất giống nhau nên không quá khó.
Dưới đây là bảng so sánh giữa hai công cụ dòng lệnh Docker và `crictl`. Như bạn thấy, rất nhiều lệnh như attach, exec, images, info, inspect, logs, ps, stats, version, v.v. hoạt động y hệt nhau, và một số lệnh để tạo, xóa, start và stop image/container cũng hoạt động tương tự. Danh sách đầy đủ các khác biệt có thể xem tại [link này](https://kubernetes.io/docs/reference/tools/map-crictl-dockercli/#retrieve-debugging-information).

| docker cli | crictl            | Mô tả                                                                 | Tính năng không hỗ trợ              |
|------------|-------------------|----------------------------------------------------------------------|-------------------------------------|
| attach     | attach            | Gắn (attach) vào một container đang chạy                              | --detach-keys, --sig-proxy          |
| exec       | exec              | Chạy một lệnh bên trong container đang chạy                           | --privileged, --user, --detach-keys |
| images     | images            | Liệt kê image                                                         |                                     |
| info       | info              | Hiển thị thông tin toàn hệ thống                                      |                                     |
| inspect    | inspect, inspecti | Trả về thông tin chi tiết (low-level) của container, image hoặc task  |                                     |
| logs       | logs              | Lấy log của container                                                 | --details                           |
| ps         | ps                | Liệt kê container                                                     |                                     |
| stats      | stats             | Hiển thị liên tục thống kê sử dụng tài nguyên của (các) container     | Column: NET/BLOCK I/O, PIDs         |
| version    | version           | Hiển thị thông tin phiên bản của runtime (Docker, ContainerD hoặc khác) |                                   |

Như đã nói, `crictl` có thể kết nối tới bất kỳ runtime tương thích CRI nào, nên hãy nhớ thiết lập đúng endpoint nếu bạn cấu hình nhiều container runtime, hoặc nếu muốn `crictl` tương tác với một runtime cụ thể. Nếu bạn không cấu hình gì, mặc định nó sẽ thử kết nối tới các socket theo thứ tự sau: dockershim trước, rồi containerd, rồi CRI-O, rồi CRI-dockerd. Nếu muốn ghi đè và chỉ định một endpoint cụ thể, bạn dùng tùy chọn `--runtime-endpoint` với lệnh `crictl`, hoặc dùng biến môi trường `CONTAINER_RUNTIME_ENDPOINT` và đặt giá trị là endpoint mong muốn.

![](../../images/02-03-09.png)

> 💡 **Bổ sung:** Ở các phiên bản `crictl` mới, dockershim đã bị loại khỏi danh sách endpoint mặc định, và việc dựa vào endpoint mặc định đã bị deprecated (crictl sẽ in cảnh báo). Cách gọn nhất là cấu hình cố định trong file `/etc/crictl.yaml`, ví dụ với containerd là dòng `runtime-endpoint: unix:///run/containerd/containerd.sock` (và tương tự `image-endpoint: ...`), hoặc dùng lệnh `crictl config --set runtime-endpoint=unix:///run/containerd/containerd.sock`. Trong môi trường thi (và lab KodeKloud), `crictl` thường đã được cấu hình sẵn. Các lệnh hay dùng khi troubleshoot node: `crictl ps -a` (xem cả container đã thoát — hữu ích khi kube-apiserver bị lỗi khiến `kubectl` không dùng được), `crictl logs <container-id>`, `crictl pods`, `crictl images`.

Tóm lại, chúng ta có tiện ích dòng lệnh `ctr` đi kèm containerd, làm việc với containerd, chỉ dùng cho mục đích debug và có tập tính năng rất hạn chế — lý tưởng nhất là bạn không cần dùng đến nó, có thể bỏ qua. Tiếp theo là CLI `nerdctl`, cũng đến từ cộng đồng containerd, nhưng là một CLI kiểu Docker dành cho containerd, dùng cho mục đích chung như tạo container và hỗ trợ tính năng tương đương hoặc nhiều hơn Docker CLI — mình nghĩ đây là công cụ sẽ được dùng nhiều hơn trong tương lai. Cuối cùng là tiện ích `crictl` đến từ cộng đồng Kubernetes, chủ yếu dùng để tương tác với các runtime tương thích CRI — không chỉ containerd mà mọi runtime hỗ trợ CRI — và một lần nữa, chủ yếu dùng cho mục đích debug.

![](../../images/02-03-10.png)

Nhìn vào bảng so sánh, bạn có thể thấy `ctr` và `crictl` chủ yếu dùng để debug, còn `nerdctl` dùng cho mục đích chung. `ctr` và `nerdctl` đến từ cộng đồng containerd và làm việc với containerd, trong khi `crictl` đến từ cộng đồng Kubernetes và hoạt động với mọi runtime tương thích CRI.
Các lab của chúng ta ban đầu cài Docker trên mọi node nên chúng ta dùng lệnh Docker để troubleshoot, nhưng giờ tất cả đều là containerd, nên hãy nhớ dùng lệnh `crictl` thay thế khi troubleshoot.


