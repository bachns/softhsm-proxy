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

## Biên dịch softhsm-server image và pkcs11-client

```
docker build -f Dockerfile.server -t softhsm-server .
docker build -f Dockerfile.client -t pkcs11-client .
```

## softhsm-server container

```
docker network create softhsm-net
docker create --name softhsm-server --network softhsm-net --publish 2345:2345 softhsm-server
docker start softhsm-server
```

## pkcs11-client container

Mở một cửa sổ terminal khác chạy client
```
docker run -d -it --name pkcs11-client --network softhsm-net pkcs11-client
```

Vào bash của client container
```
docker exec -it pkcs11-client bash 
```

Kiểm tra kết nối tới softHSM sau khi vào bash của client container
```
pkcs11-tool --show-info
pkcs11-tool --list-slots
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
