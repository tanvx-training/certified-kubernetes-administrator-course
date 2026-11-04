# Practice Test - Cluster Roles (Bài thực hành)
  - Đi tới [Practice Test](https://kodekloud.com/topic/practice-test-cluster-roles/)
 
Lời giải cho bài practice test - Cluster Roles
- Chạy lệnh kubectl get clusterroles --no-headers | wc -l hoặc kubectl get clusterroles --no-headers -o json | jq '.items | length'
  
  <details>
  
  ```
  $ kubectl get clusterroles --no-headers | wc -l (or)
  $ kubectl get clusterroles --no-headers -o json | jq '.items | length'
  ```
  
  </details>
  
- Chạy lệnh kubectl get clusterrolebindings --no-headers | wc -l hoặc kubectl get clusterrolebindings --no-headers -o json | jq '.items | length'

  <details>
  
  ```
  $ kubectl get clusterrolebindings --no-headers | wc -l (or)
  $ kubectl get clusterrolebindings --no-headers -o json | jq '.items | length'
  ```
  
  </details>
  
- ClusterRole cluster-admin thuộc namespace nào?

  <details>
  
  ```
  $ Cluster roles are cluster wide and not part of any namespace
  ```
  
  </details>

  > 💡 Đáp án: ClusterRole có phạm vi toàn cluster và không thuộc namespace nào.
  
- Chạy lệnh kubectl describe clusterrolebinding cluster-admin
  
  <details>
  
  ```
  $ kubectl describe clusterrolebinding cluster-admin
  ```
  
  </details>

  > 💡 ClusterRoleBinding cluster-admin gắn ClusterRole `cluster-admin` cho group `system:masters`.
  
- Chạy lệnh kubectl describe clusterrole cluster-admin

  <details>
  
  ```
  $ kubectl describe clusterrole cluster-admin
  ```
  
  </details>

  > 💡 Kết quả có resource `*.*` và verb `[*]`, nghĩa là ClusterRole này được thực hiện mọi thao tác trên mọi resource trong cluster.
  
- Xem đáp án tại /var/answers

  <details>
  
  ```
  $ kubectl create -f /var/answers/michelle-node-admin.yaml
  ```
  
  </details>

  > 💡 Cách làm imperative tương đương (cấp cho user michelle quyền quản lý node):
  > - `kubectl create clusterrole michelle-role --verb=get,list,watch --resource=nodes`
  > - `kubectl create clusterrolebinding michelle-role-binding --clusterrole=michelle-role --user=michelle`
  > - Kiểm tra: `kubectl auth can-i list nodes --as michelle`
  
- Xem đáp án tại /var/answers

  <details>
  
  ```
  $ kubectl create -f /var/answers/michelle-storage-admin.yaml
  ```
  
  </details>

  > 💡 Cách làm imperative tương đương (quyền trên PersistentVolume và StorageClass — đều là resource cluster-scoped):
  > - `kubectl create clusterrole storage-admin --verb=get,list,watch,create,delete --resource=persistentvolumes,storageclasses`
  > - `kubectl create clusterrolebinding michelle-storage-admin --clusterrole=storage-admin --user=michelle`
