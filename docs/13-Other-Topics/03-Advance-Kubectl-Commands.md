# Các lệnh Kubectl nâng cao

  - Xem [Bài giảng](https://kodekloud.com/topic/advanced-kubectl-commands/)

  - Lấy output của **`kubectl`** ở định dạng json: 

    ```
    kubectl get nodes -o json
    ```

    ```
    kubectl get pods -o json 
    ```

    ![pod](../../images/jpod.PNG)

    > 💡 **Bổ sung:** Xem output JSON đầy đủ là bước đầu tiên khi viết JSONPath — nó cho bạn biết chính xác "đường đi" tới trường cần lấy. Lưu ý: khi `get` nhiều object (không chỉ định tên), kết quả là một `List` nên đường dẫn bắt đầu bằng `.items[...]`; khi `get` một object cụ thể (vd `kubectl get node node01 -o json`) thì không có `.items`. Có thể dùng `kubectl explain pod.spec.containers` để tra cứu cấu trúc các trường.

  - Lấy tên image được Pod sử dụng bằng truy vấn JSONPath:

    ```
    kubectl get pods -o=jsonpath='{.items[0].spec.containers[0].image}'
    ```

  - Lấy tên các node trong cluster:

    ```
    kubectl get pods -o=jsonpath='{.items[*].metadata.name}'
    ```

    ![node](../../images/jnode.PNG)


  - Lấy kiến trúc (architecture) của các node trong cluster:

    ```
    kubectl get pods -o=jsonpath='{.items[*].status.nodeInfo.architecture}'
    ```

  - Lấy số CPU của các node trong cluster:

    ```
    kubectl get pods -o=jsonpath='{.items[*].status.status.capacity.cpu}'
    ```

  > 💡 **Bổ sung — đính chính các lệnh trên:** Ba lệnh cuối trong bản gốc bị viết nhầm `pods` thay vì `nodes` (trường `nodeInfo`, `capacity` thuộc về Node). Lệnh đúng là:
  > - `kubectl get nodes -o=jsonpath='{.items[*].metadata.name}'`
  > - `kubectl get nodes -o=jsonpath='{.items[*].status.nodeInfo.architecture}'`
  > - `kubectl get nodes -o=jsonpath='{.items[*].status.capacity.cpu}'` (chỉ có một `.status`, không phải `.status.status`)
  >
  > Có thể ghép nhiều truy vấn và thêm ký tự xuống dòng/tab: `kubectl get nodes -o=jsonpath='{.items[*].metadata.name}{"\n"}{.items[*].status.capacity.cpu}{"\n"}'`.

  #### Vòng lặp - Range

  > 💡 **Bổ sung:** Kết quả của `{.items[*]...}` in tất cả giá trị trên một dòng, rất khó đọc. Để in mỗi node một dòng (tên node và số CPU), dùng vòng lặp `range ... end` của JSONPath:
  >
  > `kubectl get nodes -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.capacity.cpu}{"\n"}{end}'`
  >
  > Mọi thứ nằm giữa `{range .items[*]}` và `{end}` được lặp lại cho từng phần tử, `{"\t"}` là tab và `{"\n"}` là xuống dòng.

  - Một cách đơn giản hơn để in output thành các cột riêng biệt (một cột là tên node, cột kia là số CPU) là dùng **custom-columns**:

    ```
    kubectl get nodes -o=custom-columns=NODE:.metadata.name ,CPU:.status.capacity.cpu
    ```
    
    ![loop](../../images/loop.PNG)

    > 💡 **Bổ sung:** Cú pháp là `-o=custom-columns=<TÊN-CỘT>:<đường-dẫn-JSON>,<TÊN-CỘT>:<đường-dẫn>`. **Không được có dấu cách** trước/sau dấu phẩy (lệnh trong bản gốc có một dấu cách trước `,CPU` sẽ gây lỗi), đúng là: `kubectl get nodes -o=custom-columns=NODE:.metadata.name,CPU:.status.capacity.cpu`. Với custom-columns, đường dẫn tính từ **từng object** nên không cần `.items[*]` và không cần dấu `{}`. Với mảng có thể dùng `.spec.containers[*].image` hoặc `.spec.containers[0].image`.

  - Kubectl có sẵn tùy chọn **`--sort-by`**, có thể kết hợp với truy vấn JSONPath để **`sắp xếp`** theo tên hoặc theo **`số CPU`**

    ```
    kubectl get nodes --sort-by=.metadata.name
    ```

    ![loop](../../images/loop.PNG)

    ```
    kubectl get nodes --sort-by=..status.capacity.cpu
    ```

    > 💡 **Bổ sung:** Lệnh cuối trong bản gốc thừa một dấu chấm; viết chuẩn là `kubectl get nodes --sort-by=.status.capacity.cpu`. `--sort-by` cũng tính đường dẫn từ từng object (không cần `.items[*]`) và có thể kết hợp với custom-columns, ví dụ: `kubectl get pv --sort-by=.spec.capacity.storage -o=custom-columns=NAME:.metadata.name,CAPACITY:.spec.capacity.storage`.
    >
    > Lọc phần tử trong mảng bằng biểu thức điều kiện `?()`, ví dụ lấy địa chỉ InternalIP của các node: `kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'`. Trong phòng thi, tài liệu chính thức "JSONPath Support" trên kubernetes.io và mục "kubectl Quick Reference" (cheat sheet) rất hữu ích để tra cứu nhanh.
