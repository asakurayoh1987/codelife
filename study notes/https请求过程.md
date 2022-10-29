# Https请求握手过程

## 前置知识

### 1. 非对称加密——公钥与私钥

* 传统对称加密，双方使用相同的key，如果key泄漏则所有通信的双方，其消息都失去了安全保障
* 非对称加密，key分为公钥与私钥，公钥加密的消息只有私钥可以解，反之亦然，私钥自己保留，仅对外公布公钥

### 1.1 性能问题

* 非对称加密虽然安全，但存在性能问题
* 并非对整个消息进行非对称加密，而且是利用非对称加密来安全传输对称加密的密钥，后续的消息通信依旧是使用对称加密

### 2. 中间人攻击

* A(发送方)-->C(中间人)-->B(接收方)
* 中间人也有一套自己的私钥与公钥，请求方拿到的是中间人的公钥，中间人拿到消息后用自己的私钥解密，然后再用接收方的公钥加密进行传递

### 3. 数字证书（身份确认）

* hash(公钥+身份信息)=>信息摘要(虽不可逆，但可以被整个替换)
* CA的私钥加密(信息摘要)=>数字签名
* 信息摘要+数字签名=>数字证书
* hash(公钥+身份信息)===CA的公钥解密(数字签名)，通过此来确认身份（于是要解决的问题就是CA是否可信）

CA是否可信？
1. CA也有证书
2. CA的信用像树一样分组，高层的CA可以证明低层CA的信用
3. 顶层CA预置在操作系统和浏览器当中
4. 也可以手动将CA设为可信，比如自己生成的CA证书

### 4. 关键参数

* client_random：客户端随机数
* server_random：服务端随机数
* premaster_secret：预主密钥，生成正式密钥前的一个密钥

### 5. 密钥交换(握手)算法：RSA vs ECDH

#### 5.1 RSA

RSA与Diffie-HellmanF最大的不同在于预主密钥的生成，前者是在客户端生成，并通过公钥加密传递给服务端，服务端通过私钥解密拿到，也就是说私钥参与了密钥交换的过程，如果私钥泄漏了，就可以计算出预主密钥，再结合以前的报文，就可以解密传递的消息，所以RSA不是前向安全的（perfect forward secrecy--PFS）。

![https_process](/Users/ycyu/Documents/Typora Notes/media/ssl_rsa_shake.png)

#### 5.3 ECDH(Elliptic Curve Diffie-Hellman)

ECDH在每次握手时都会生成临时的密钥对，即预主密钥并在网络中传递，即使私钥被破解，之前的历史消息并不会收到影响，所以是前向安全的（perfect forward secrecy--PFS）。在TLS1.3中ECDHE彻底代替的RSA。

![https_process](/Users/ycyu/Documents/Typora Notes/media/ssl_ecdhe_shake.png)

## wireshark抓包分析

### 1. 完整过程

![https_process](/Users/ycyu/Documents/Typora Notes/media/https_process.png)

![https_process](/Users/ycyu/Documents/Typora Notes/media/https_process_2.png)

### 2. 分步骤分析

#### 2.1 client hello（c->s）

![https_process](/Users/ycyu/Documents/Typora Notes/media/https_client_hello.png)

传递client_random，支持的加密套件

#### 2.2 server hello（s->c）

![https_process](/Users/ycyu/Documents/Typora Notes/media/https_server_hello.png)

传递server_random，以及选择的加密套件为`Cipher Suite: TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (0xc02f)`，其中：

* TLS：基于TLS协议
* ECDHE：密钥交换的非对称加密算法(ECDHE椭圆曲线算法)
* RSA：证书校验算法
* AES_128_GCM：对称加密算法，使用128位AES加密，使用GCM分组模式
* SHA256：信息摘要的哈希算法
* 0xc02f：加密套件的标识

#### 2.3 certificate（s->c）

传递数字证书给客户端

![https_process](/Users/ycyu/Documents/Typora Notes/media/https_certificate.png)

#### 2.4 server_key_exchange（s->c）

![https_process](/Users/ycyu/Documents/Typora Notes/media/https_server_key_exchange.png)

#### 2.5 server_hello_done（s->c）

表示server阶段处理结束

#### 2.5 client_key_exchange, change_cipher_spec, encrypted_handshake_message

![https_process](/Users/ycyu/Documents/Typora Notes/media/https_client_key_exchange.png)

#### 2.6 change_cipher_spec

#### 2.7 encrypted_handshake_message

