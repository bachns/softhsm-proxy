# Hướng dẫn cài đặt SoftHSM Server

## Cài docker và podman

```
yum install docker podman
```

## Cài SotHSM Server

Copy tệp tin softhsm-server.tar vào /root sau đó chạy các lệnh sau:

```
podman load --input /root/softhsm-server.tar
podman run -d -p 2345:2345 --name softhsm-server localhost/softhsm-server
```

Xác nhận softhsm-server đã chạy ở cổng 2345 bằng lệnh sau:

```
netstat -an | grep 2345
```


# Hướng dẫn cài đặt PKCS11 Client

## Cài đặt các gói cần thiết

```
yum update && yum install -y gcc gcc-c++ kernel-devel make cmake git \
    openssl-devel libseccomp-devel gnutls-utils opensc
```

## Cài đặt pkcs11 proxy

```
git clone https://github.com/SUNET/pkcs11-proxy \
    && cd pkcs11-proxy && mkdir build && cd build && cmake .. && make && make install 
```

## Cài đặt biến môi trường

Copy tệp tin tlskey.psk vào /root/tlskey.psk sau đó đặt các biến môi trường sau:

```
export PKCS11_PROXY_TLS_PSK_FILE="/root/tlskey.psk"
export PKCS11_PROXY_SOCKET="tls://<ip-sorthsm-server>:2345"
```

## Truy cập tới SoftHSM Server

Xem các tokens:

```
p11tool --provider /usr/local/lib/libpkcs11-proxy.so --list-tokens
```

Xem cặp khóa và cert (password: 123456):

```
p11tool --provider /usr/local/lib/libpkcs11-proxy.so --login --list-all "TOKEN_URL"
```