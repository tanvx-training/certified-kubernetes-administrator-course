# Kiến thức cơ bản về TLS (TLS Basics)
  - Xem [Video hướng dẫn](https://kodekloud.com/topic/tls-basics/)
  
Trong phần này, chúng ta sẽ tìm hiểu các kiến thức cơ bản về TLS.

## Certificate
- Certificate (chứng chỉ) được dùng để đảm bảo sự tin cậy giữa 2 bên trong một giao dịch.
- Ví dụ: khi user truy cập vào một web server, TLS certificate đảm bảo rằng kết nối giữa hai bên được mã hóa, và user đang nói chuyện đúng với server mà họ muốn.

  ![cert1](../../images/cert1.PNG)
  
  
## Mã hóa đối xứng (Symmetric Encryption)
- Đây là một cách mã hóa an toàn, nhưng nó dùng cùng một key để mã hóa và giải mã dữ liệu. Key này phải được trao đổi giữa bên gửi và bên nhận qua mạng, nên có nguy cơ hacker lấy được key và giải mã dữ liệu.

  ![cert2](../../images/cert2.PNG)
  
## Mã hóa bất đối xứng (Asymmetric Encryption)
- Thay vì dùng một key duy nhất để mã hóa và giải mã, mã hóa bất đối xứng dùng một cặp key: private key và public key. Dữ liệu được mã hóa bằng public key chỉ có thể giải mã bằng private key tương ứng.

  ![cert3](../../images/cert3.PNG)
  
  ![cert4](../../images/cert4.PNG)
  
  ![cert5](../../images/cert5.PNG)
  
  ![cert6](../../images/cert6.PNG)

> 💡 **Bổ sung:** Trong thực tế TLS kết hợp cả hai: mã hóa bất đối xứng chỉ dùng trong giai đoạn bắt tay (handshake) để xác thực server và trao đổi an toàn một session key; sau đó toàn bộ dữ liệu được mã hóa bằng session key đó theo kiểu đối xứng (nhanh hơn nhiều). Ví dụ quen thuộc của cặp key bất đối xứng là SSH: tạo bằng `ssh-keygen`, public key (`id_rsa.pub`) được đặt vào `~/.ssh/authorized_keys` trên server, còn private key (`id_rsa`) giữ bí mật ở client.
  

#### Làm thế nào để xem một certificate và kiểm tra xem nó có hợp lệ không?
- Xem ai là người ký và cấp certificate đó.
- Nếu bạn tự tạo certificate thì bạn cũng phải tự ký cho nó; đó được gọi là self-signed certificate (certificate tự ký). Trình duyệt sẽ cảnh báo không tin cậy với loại certificate này.

  ![cert7](../../images/cert7.PNG)
  
#### Làm sao để tạo một certificate hợp lệ? Làm sao để certificate được ký bởi một bên có thẩm quyền?
- Đó là lúc **`Certificate Authority (CA)`** vào cuộc. Một số CA phổ biến là Symantec, DigiCert, Comodo, GlobalSign, v.v. Bạn tạo một CSR (Certificate Signing Request) gửi tới CA; CA xác minh thông tin rồi ký và trả lại certificate. Trình duyệt có sẵn public key của các CA phổ biến nên có thể kiểm tra chữ ký đó.

  ![cert8](../../images/cert8.PNG)
  
  ![cert9](../../images/cert9.PNG)
  
  ![cert10](../../images/cert10.PNG)
  
## Public Key Infrastructure (PKI)
   
   ![pki](../../images/pki.PNG)

- Toàn bộ hạ tầng gồm CA, server, client, các cặp key và certificate cùng quy trình tạo, phân phối, quản lý chúng được gọi là PKI.
   
## Quy ước đặt tên file certificate

  ![cert11](../../images/cert11.PNG)

> 💡 **Bổ sung:** Mẹo nhận biết nhanh: file certificate (chứa public key) thường có đuôi `.crt` hoặc `.pem` (vd `server.crt`, `server.pem`, `client.pem`); file private key thường có đuôi `.key` hoặc tên chứa chữ `key` (vd `server.key`, `server-key.pem`). Cặp `apiserver.crt` / `apiserver.key` trong `/etc/kubernetes/pki` là ví dụ điển hình.
