# Practice Test - Các lệnh Kubectl nâng cao (Bài thực hành)

  - Đi tới [Practice Test nâng cao về các lệnh Kubectl](https://kodekloud.com/topic/practice-test-advanced-kubectl-commands/)

  ### Lời giải

   1. Xem lời giải 

       <details>
       
        ```
        kubectl get nodes -o json > /opt/outputs/nodes.json
        ```   
       </details>

   2. Xem lời giải 

       <details>

        ```
        kubectl get node node01 -o json > /opt/outputs/node01.json
        ```   
       </details>

   3. Xem lời giải

       <details>

        ```
        kubectl get nodes -o=jsonpath='{.items[*].metadata.name}' > /opt/outputs/node_names.txt
        ```
       </details>

   4. Xem lời giải

       <details>

        ```
        kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/outputs/nodes_os.txt
        ```

        > 💡 **Giải thích:** Nếu không biết tên trường, hãy chạy `kubectl get nodes -o json | less` (hoặc xem file `nodes.json` vừa tạo ở câu 1) và tìm từ khóa `os` — thông tin hệ điều hành nằm trong `.status.nodeInfo` (gồm `osImage`, `kernelVersion`, `kubeletVersion`, `containerRuntimeVersion`, `architecture`...).
       </details>

   5. Xem lời giải

       <details>

        ```
        kubectl config view --kubeconfig=my-kube-config -o jsonpath="{.users[*].name}" > /opt/outputs/users.txt
        ```

        > 💡 **Giải thích:** `kubectl config view` cũng hỗ trợ `-o jsonpath`. Cấu trúc của kubeconfig có ba mảng chính: `clusters`, `users`, `contexts`; vì đây là một object duy nhất (không phải List) nên không có `.items`. Tùy chọn `--kubeconfig` chỉ định file kubeconfig cần đọc thay cho `~/.kube/config`.
       </details>

   6. Xem lời giải

       <details>

        ```
        kubectl get pv --sort-by=.spec.capacity.storage > /opt/outputs/storage-capacity-sorted.txt
        ```
       </details>

   7. Xem lời giải

       <details>

        ```
        kubectl get pv --sort-by=.spec.capacity.storage -o=custom-columns=NAME:.metadata.name,CAPACITY:.spec.capacity.storage > /opt/outputs/pv-and-capacity-sorted.txt
        ```

        > 💡 **Giải thích:** Tên cột (`NAME`, `CAPACITY`) phải viết đúng như đề bài yêu cầu. Nhớ không có dấu cách quanh dấu phẩy trong `custom-columns`.
       </details>

   8. Xem lời giải

       <details>

        ```
        kubectl config view --kubeconfig=my-kube-config -o jsonpath="{.contexts[?(@.context.user=='aws-user')].name}" > /opt/outputs/aws-context-name
        ```

        > 💡 **Giải thích:** `?()` là biểu thức lọc (filter) của JSONPath, còn `@` đại diện cho phần tử hiện tại trong mảng. Biểu thức trên nghĩa là: trong mảng `contexts`, chọn những phần tử có `context.user` bằng `aws-user`, rồi lấy `name` của chúng. Vì toàn bộ biểu thức đặt trong dấu nháy kép `"..."`, giá trị so sánh bên trong dùng dấu nháy đơn `'aws-user'`.
       </details>
       
       
       