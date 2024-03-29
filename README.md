# SoftHSM and PKCS11-Proxy

Cài đặt SoftHSM và PKCS11-Proxy với Docker. Chúng ta sẽ tạo một máy chủ softhsm-server chạy softHSM và một máy khách pkcs11-client sử dụng pkcs11-proxy để gọi đến softHSM nằm trên máy chủ.

## Yêu cầu

Máy tính cần cài đặt git và docker, đồng thời dịch vụ docker đang chạy nền. Có thể mở terminal để kiểm tra cài đặt bằng các lệnh sau:

```
git --version 
docker --version
```

## Chuẩn bị

```
git clone https://github.com/bachns/softhsm-proxy.git
cd softhsm-proxy
```

## Biên dịch softhsm-server image và pkcs11-client image

```
docker build -f Dockerfile.server -t bachns/softhsm-server .
docker build -f Dockerfile.client -t bachns/pkcs11-client .
```

## softhsm-server container

```
docker network create softhsm-net
docker create --network softhsm-net --name softhsm-server --hostname softhsm-server -p 2345:2345 bachns/softhsm-server
docker start softhsm-server
```

## pkcs11-client container

Mở một cửa sổ terminal khác chạy client
```
docker run -it --name pkcs11-client --network softhsm-net  bachns/pkcs11-client
```

Kiểm tra kết nối tới softHSM sau khi vào bash của client container.
Nếu kết quả xuất hiện token vgca_example_token là đã truy cập vào SoftHSM thành công.
```
p11tool --provider /usr/local/lib/libpkcs11-proxy.so --list-tokens
```

Tạo cặp khóa
```
pkcs11-tool --module /usr/local/lib/libpkcs11-proxy.so --login --login-type user --keypairgen --id "YOUR_KEY_ID" --label "YOUR_KEY_LABEL" --key-type rsa:3072
```

Xem cặp khóa vừa tạo và URL của các khóa
```
p11tool --provider /usr/local/lib/libpkcs11-proxy.so --login --list-all "TOKEN_URL"
```

Tạo certificate signing request (CSR)
```
certtool --provider=/usr/local/lib/libpkcs11-proxy.so --generate-request --load-pubkey "PUBLIC_KEY_URL" --load-privkey "PRIVATE_KEY_URL" --outfile request.csr 
```

Sao chép tệp request.csr từ container sang máy host
```
docker cp <container_id>:/path/to/request.csr .
```

Sao chép cert đã được cấp từ host vào container
```
docker cp /path/to/<your_cert>.crt <container_id>:/path/to/<your_cert>.crt
```

Thực hiện nạp cert vào SoftHSM
```
pkcs11-tool --module /usr/local/lib/libpkcs11-proxy.so --login --pin 123456 --write-object "YOUR_CERT_FILE" --type cert --id "YOUR_KEY_ID"
```

Kiểm tra cert vừa nạp
```
p11tool --provider /usr/local/lib/libpkcs11-proxy.so --login --list-all "TOKEN_URL"
```

## Dừng container

```
docker stop softhsm-server
docker stop pkcs11-client
```

## Khởi động lại container

```
docker start softhsm-server
docker start pkcs11-client
```
