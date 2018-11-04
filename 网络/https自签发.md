
### 一、创建 CA

#### 1、生成 CA 私钥
````
openssl genrsa -out ca.key 1024
````

#### 2、生成 CA 证书请求文件
````
openssl req -new -key ca.key -out ca.csr
````

#### 3、生成 CA 证书
````
openssl x509 -req -in ca.csr -signkey ca.key -days 365 -out ca.crt
````

### 二、创建服务端证书

#### 1、生成服务端私钥
````
openssl genrsa -out server.key 1024
````

#### 2、生成服务端证书请求文件
````
openssl req -new -key server.key -out server.csr
````

#### 3、CA机构颁发服务器证书
````
openssl x509 -req -sha256 -extfile v3.ext -CA ca.crt -CAkey ca.key -CAcreateserial -in server.csr -days 365 -out server.crt
````

### 三、v3.ext 文件
````
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage=digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName=@alt_names

[alt_names]
DNS.1=www.test.com
````

### 四、参考
- https://yi-love.github.io/articles/https-ca
- https://gist.github.com/prof3ssorSt3v3/fab15b677d4a4cc2568f09d477d9c8ac
