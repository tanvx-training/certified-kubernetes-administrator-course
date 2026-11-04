# Cài đặt Terraform

Ở bước này, chúng ta sẽ cài đặt terraform và tải cấu hình sẽ dùng về từ Github.

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

> 💡 **Bổ sung:** Đoạn lệnh trên lấy số phiên bản terraform mới nhất từ API của HashiCorp, tải file zip tương ứng cho Linux amd64, giải nén và chuyển file thực thi `terraform` vào `~/bin` (thư mục này nằm trong `PATH` của Cloud Shell). Lệnh cuối `terraform version` dùng để kiểm tra cài đặt thành công.

# Clone repository

```bash
git clone https://github.com/kodekloudhub/certified-kubernetes-administrator-course.git
cd certified-kubernetes-administrator-course/managed-clusters/aks/terraform
```

Tiếp theo: [Triển khai Cluster](./04-deploy-cluster.md)
