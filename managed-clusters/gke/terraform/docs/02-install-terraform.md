# Cài đặt Terraform

Ở bước này, chúng ta sẽ cài đặt terraform và tải cấu hình sẽ dùng về từ Github.

## Mở Cloudshell

Trước tiên, mở Cloudshell bằng cách bấm vào biểu tượng ở góc trên bên phải của dashboard. Một terminal Linux sẽ mở ra ở phía dưới màn hình.

![image](../images/02-cloudshell.png)

Thực hiện các lệnh sau trong terminal Cloudshell

## Cài đặt terraform

```bash
terraform_version=$(curl -s https://checkpoint-api.hashicorp.com/v1/check/terraform | jq -r -M '.current_version')
curl -O "https://releases.hashicorp.com/terraform/${terraform_version}/terraform_${terraform_version}_linux_amd64.zip"
unzip terraform_${terraform_version}_linux_amd64.zip
mkdir -p ~/bin
mv terraform ~/bin/
terraform version
```

> 💡 **Bổ sung:** Đoạn lệnh trên lấy số phiên bản terraform mới nhất từ API của HashiCorp, tải file zip tương ứng cho Linux amd64, giải nén và chuyển file thực thi `terraform` vào `~/bin`. Google Cloud Shell thường đã cài sẵn một phiên bản terraform; các lệnh trên đảm bảo bạn dùng bản mới nhất. Nếu `terraform version` vẫn hiển thị bản cũ, hãy kiểm tra `~/bin` có nằm trong `PATH` hay không (`echo $PATH`).

# Clone repository

```bash
git clone https://github.com/kodekloudhub/certified-kubernetes-administrator-course.git
cd certified-kubernetes-administrator-course/managed-clusters/gke/terraform
```

Tiếp theo: [Triển khai Cluster](./03-deploy-cluster.md)</br>
Trước: [Đăng nhập](./01-sign-in.md)
