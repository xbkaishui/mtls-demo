# openssl 证书使用

## 生成 CA 私钥
openssl genrsa -out ca.key 4096

## 生成CA 的数字证书
openssl req -new -x509 -days 3650 -key ca.key -out ca.crt

## 生成 server 端的私钥
openssl genrsa -out server.key 4096

## 生成 server 端数字证书请求
openssl req -new -key server.key -out server.csr

- 注意
Common Name (e.g. server FQDN or YOUR name) []
如果使用ip地址访问的，所以Common Name，输入ip即可。
如果使用域名访问，那么这一步，必须是域名才行！

## 用CA私钥签发server的数字证书
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 3650

## 生成客户端的私钥与证书
openssl genrsa -out client.key 4096

## 生成 client 端数字证书请求
openssl req -new -key client.key -out client.csr

## 用CA私钥签发 client 的数字证书
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -days 3650

## 配置nginx
```
server {
        listen 443;
        server_name localhost;
        ssl on;
        ssl_certificate /etc/nginx/keys/server.crt;#配置证书位置
        ssl_certificate_key /etc/nginx/keys/server.key;#配置秘钥位置
        ssl_client_certificate /etc/nginx/keys/ca.crt;#双向认证
        ssl_verify_client on; #双向认证
        ssl_session_timeout 5m;
        ssl_protocols SSLv2 SSLv3 TLSv1 TLSv1.1 TLSv1.2; #按照这个协议配置
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE; #按照这个套件配置
        ssl_prefer_server_ciphers on;
        root html;
        index index.html;
        location / {
                try_files $uri $uri/ =404;
        }
}
```

## 查看KEY信息

> openssl rsa -noout -text -in myserver.key

## 查看CSR信息

> openssl req -noout -text -in myserver.csr

## 查看证书信息

> openssl x509 -noout -text -in ca.crt

## 验证证书, 会提示self signed
openssl verify -CAfile ca.crt server.crt





### 文件区别

https://blog.freessl.cn/ssl-cert-format-introduce/

根据不同的服务器以及服务器的版本，我们需要用到不同的证书格式，就市面上主流的服务器来说，大概有以下格式：

.DER .CER，文件是二进制格式，只保存证书，不保存私钥
.PEM，一般是文本格式，可保存证书，可保存私钥
Privacy Enhanced Mail，一般为文本格式，以 -----BEGIN... 开头，以 -----END... 结尾。中间的内容是 BASE64 编码。这种格式可以保存证书和私钥，有时我们也把PEM 格式的私钥的后缀改为 .key 以区别证书与私钥。具体你可以看文件的内容。
这种格式常用于 Apache 和 Nginx 服务器。

.CRT，可以是二进制格式，可以是文本格式，与 .DER 格式相同，不保存私钥

.PFX .P12，二进制格式，同时包含证书和私钥，一般有密码保护
.JKS，二进制格式，同时包含证书和私钥，一般有密码保护

各种文件之间相互转换

https://blog.csdn.net/zwc609906/article/details/93887339