# Câu hỏi về trạng thái cluster (Cluster State Questions)

Dạng câu này xuất hiện trong cả CKA lẫn CKAD: đề yêu cầu ghi ra file, ví dụ, Pod tiêu thụ nhiều CPU hoặc nhiều memory nhất, hoặc các dạng khác như liệt kê IP của Pod (thường dùng jsonpath hoặc custom columns). Những câu này hay bị chấm sai khi kết thúc bài thi và khiến nhiều học viên rất bối rối.

Những bạn tinh ý có thể đã nhận ra lý do. Nếu gặp câu như vậy ở đầu bài mock, và các câu sau lại yêu cầu bạn deploy thêm vào cùng cluster đó, thì kết quả sẽ thay đổi! Pod đang dùng nhiều CPU nhất lúc bạn trả lời có thể không còn đứng đầu khi kết thúc bài thi, vì một Pod mới deploy có thể dùng nhiều CPU hơn.

Dạng câu này liên quan đến trạng thái cluster, và trạng thái đó thay đổi mỗi khi bạn deploy hoặc xóa resource. Script chấm điểm chỉ có thể xét trạng thái cluster sau khi bạn bấm `End Exam`. Mẹo ở đây là hoãn trả lời những câu như vậy cho tới khi sắp kết thúc bài thi. Lý tưởng nhất (dù không phải lúc nào cũng làm được) là viết một script để trả lời câu hỏi và chạy thử ngay khi đang ở câu đó. Sau đó, khi chuẩn bị bấm nút `End Exam`, chạy lại script để cập nhật file theo trạng thái hiện tại. Như vậy câu hỏi sẽ được chấm đúng. Trong kỳ thi thật, có lẽ bạn không cần làm vậy vì nhiều khả năng sẽ có một cluster dành riêng cho các câu dạng này nên trạng thái của nó không đổi cho tới cuối bài thi.

## Ví dụ

### Ví dụ 1

Lưu `tên Pod` và `địa chỉ IP` của chúng ở tất cả namespace vào `/root/pod_ips_ckad02_svcn`, output được sắp xếp theo IP.

Đảm bảo đúng định dạng như bên dưới:
```
POD_NAME        IP_ADDR
pod-1           ip-1
pod-3           ip-2
pod-2           ip-3
...
```
---
Nhìn vào output yêu cầu, rõ ràng câu này cần dùng Custom Columns

1. Viết lệnh custom columns để ra được output như yêu cầu

    Chú ý việc dùng tham số `--context` ở đây. Nó đảm bảo lệnh chạy trên đúng cluster, bất kể bạn đã chạy `kubectl config use-context` hay chưa.

    ```
    kubectl --context=cluster3 get pods -A -o custom-columns="POD_NAME:.metadata.name,IP_ADDR:.status.podIP" --sort-by=".status.podIP"
    ```

1.  Sửa lệnh để ghi ra file và kiểm tra output

    ```
    kubectl --context=cluster3 get pods -A -o custom-columns="POD_NAME:.metadata.name,IP_ADDR:.status.podIP" --sort-by=".status.podIP" > /root/pod_ips_ckad02_svcn
    ```

    Kiểm tra
    ```bash
    cat /root/pod_ips_ckad02_svcn
    ```

1.  Giờ dùng `vi` để tạo file `run-at-end.sh`

    ```
    vi run-at-end.sh
    ```

    Dán toàn bộ lệnh kubectl ở trên (bước 2) vào file này. Nếu bạn đã tạo script này cho một câu tương tự trước đó, chỉ cần thêm dòng này vào file, để khi chạy script sẽ trả lời tất cả các câu dạng này.

1.  Chạy thử

    ```
    rm -f /root/pod_ips_ckad02_svcn
    source run-at-end.sh
    cat /root/pod_ips_ckad02_svcn
    ```

    Output phải giống như trước

1.  Cuối cùng, khi đã làm xong và trước khi bấm `End Exam`, chạy lại script

    ```
    source run-at-end.sh
    ```

### Ví dụ 2

Tìm Pod tiêu thụ nhiều CPU nhất và lưu kết quả vào file `/opt/high_cpu_pod` theo định dạng sau<br/>`cluster_name,namespace,pod_name`.

Pod có thể nằm ở bất kỳ namespace nào, trong bất kỳ cluster nào đang được cấu hình trên student-node.

---

Vì đề nói "trong bất kỳ cluster nào", câu này thực sự kiểm tra kỹ năng bash scripting của bạn; thêm nữa `kubectl top` không có tùy chọn output JSON nên càng khó viết script. Dù vậy, cách tốt nhất là ghi yêu cầu vào notepad rồi trả lời thủ công ở cuối, trước khi bấm `End Exam`. Lưu ý là bạn không cần quay lại câu hỏi để trả lời - chỉ cần làm theo ghi chú.

Áp dụng cách tương tự dù chỉ số là CPU hay memory, và resource là Pod hay Node.

* Cách thủ công
    1. Lấy tên tất cả các cluster

        ```
        kubectl config get-contexts -o name
        ```

    1.  Xem mức sử dụng tài nguyên của Pod trên từng cluster. Chạy lệnh này với từng giá trị của `--context`

        ```
        kubectl --context=cluster1 top pods -A --sort-by=cpu
        ```

    1. Khi đã xác định được Pod đứng đầu trên tất cả các cluster, tạo file output bằng vi và tự điền thông tin theo đúng định dạng yêu cầu.

* Cách dùng script

    Lưu ý - Làm theo cách này có lẽ sẽ tốn nhiều thời gian hơn bạn muốn, trừ khi bạn đã là cao thủ shell scripting!

    ```bash
    for ctx in $(kubectl config get-contexts -o name)
    do
        kubectl --context=$ctx top pod --no-headers -A --sort-by=cpu | head -1 | awk -v ctx=$ctx '{printf "%s,%s,%s,%s\n", ctx, $1, $2, $3}'
    done | sort -t ',' -k4 -h | tail -1 | sed -E 's/,[0-9]+[a-z]*$//i' > /opt/high_cpu_pod
    ```

    Khá nhiều thứ diễn ra ở đây, phải không?

    Là một DevOps engineer đi làm thực tế, đây là kiểu việc bạn sẽ được kỳ vọng tự nghĩ ra trong công việc hằng ngày - thực tế kỹ sư viết script chấm điểm cho lab này cũng phải dùng thứ gì đó tương tự! Vì vậy, để thành công trong công việc liên quan Kubernetes, việc biết *sử dụng* Linux cũng quan trọng như biết Kubernetes. Bạn không cần biết tới mức Sys Admin (ví dụ RHCSA, LFCS).<br/>Các khóa học được khuyến nghị:
    * [Linux Basics](https://kodekloud.com/courses/the-linux-basics-course/)
    * [Shell Scripts for Beginners](https://kodekloud.com/courses/shell-scripts-for-beginners/)
    * [Advanced Bash Scripting](https://kodekloud.com/courses/advanced-bash-scripting/)

    Vậy thực ra điều gì đang diễn ra?

    1. Vòng lặp `for` duyệt lần lượt từng context của cluster, lưu tên cluster vào biến `ctx`
    1. Với mỗi context, lệnh `kubectl top pods` được chạy với `-A` để lấy tất cả namespace...
        1. `--no-headers` bỏ dòng tiêu đề cột khỏi output.
        1. `--sort-by=cpu` đảm bảo Pod ta cần trong cluster này nằm ở dòng đầu tiên. Trong `kubectl top`, thứ tự sắp xếp là giảm dần.
        1. Sau đó pipe output sang `head -1` để chỉ lấy dòng đầu tiên (Pod đứng đầu của cluster này).
        1. Rồi pipe sang `awk` để định dạng output gần giống yêu cầu, truyền tên cluster vào để đưa vào output. Output sẽ trông như sau
            ```
            cluster1,default,frontend-stable-cka05-arch,396m
            ```
    1. Sau `done`, mỗi cluster sẽ có một dòng như trên. Kết quả trông như sau - lưu ý chúng đang theo thứ tự cluster, chưa theo mức dùng CPU:
        ```text
        cluster1,default,frontend-stable-cka05-arch,396m
        cluster2,kube-system,kube-apiserver-cluster2-controlplane,43m
        cluster3,kube-system,metrics-server-7b67f64457-9cqrd,5m
        cluster4,kube-system,kube-apiserver-cluster4-controlplane,32m
        ```
    1. Pipe sang `sort` để đưa Pod dùng nhiều CPU nhất *trên tất cả cluster* xuống cuối danh sách. `sort` sắp xếp tăng dần.
        1. `-t ','` đặt ký tự phân tách trường là dấu phẩy.
        1. `-k4` nghĩa là sắp xếp theo trường thứ tư (trường chứa giá trị CPU).
        1. `-h` nghĩa là sắp xếp kiểu "human" (dễ đọc cho người), có tính đến đơn vị đi kèm (ví dụ `m` cho milli-cpu). Output bây giờ trông như sau:
            ```text
            cluster3,kube-system,metrics-server-7b67f64457-9cqrd,5m
            cluster4,kube-system,kube-apiserver-cluster4-controlplane,32m
            cluster2,kube-system,kube-apiserver-cluster2-controlplane,43m
            cluster1,default,frontend-stable-cka05-arch,396m
            ```

    1. Pipe sang `tail -1` để lấy dòng cuối của danh sách đã sắp xếp - chính là dòng ta cần, kết quả là
        ```
        cluster1,default,frontend-stable-cka05-arch,396m
        ```
    1. Cuối cùng pipe sang `sed` để bỏ giá trị CPU và chỉ giữ 3 trường đầu như đề yêu cầu. Biểu thức `sed` (dùng extended regex `-E`) khớp với: dấu phẩy, theo sau là một hoặc nhiều chữ số, rồi không hoặc nhiều chữ cái, rồi tới cuối dòng; và thay bằng chuỗi rỗng, tức là xóa phần khớp. Kết quả ra đúng output yêu cầu:
        ```
        cluster1,default,frontend-stable-cka05-arch
        ```
        Sau đó redirect output vào file được yêu cầu.

> 💡 **Bổ sung:**
> - `kubectl top` cần metrics-server chạy trong cluster; nếu báo lỗi `Metrics API not available` thì kiểm tra `kubectl get pods -n kube-system | grep metrics-server`. Số liệu cũng có độ trễ khoảng vài chục giây sau khi Pod khởi động.
> - Các field thường dùng với `--sort-by`/custom columns: `.metadata.name`, `.metadata.namespace`, `.status.podIP`, `.spec.nodeName`, `.metadata.creationTimestamp`. Dùng `kubectl explain pod.status` để tìm field khi quên.
> - Khi đề yêu cầu định dạng output cụ thể, luôn `cat` file kết quả để so sánh từng ký tự (dấu phẩy, khoảng trắng, dòng tiêu đề) với mẫu trong đề.
