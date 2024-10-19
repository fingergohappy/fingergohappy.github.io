---
title: tls加密简介
cover: https://source.unsplash.com//1200x628
toc: true
date: 2024-10-19 17:17:15
tags: 
    - tls
    - https
    - ssl
categories: tls
---

啥是tls加密，啥是非对称加密，啥是对称加密，啥是数字签名，啥是证书，这些都是什么鬼？


<!--more-->

# 使用openssl 生成证书

如何使用`openssl`生成证书，这里以生成自签名证书为例。

```bash

# 生成ca私钥
openssl genpkey -algorithm RSA -out ca.key -aes256

# 生成自签名根证书
openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 -out ca.crt

# 生成服务器私钥
openssl genpkey -algorithm RSA -out server.key -aes256

# 生成证书签名请求（CSR）
openssl req -new -key server.key -out server.csr

# 使用ca证书签名服务器证书
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 3650 -sha256


# 生成客户端私钥
openssl genpkey -algorithm RSA -out client.key

# 生成客户端证书签名请求 (CSR)
openssl req -new -key client.key -out client.csr

# 使用ca证书签名客户端证书
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -days 365 -sha256

# 生成客户端p12证书
openssl pkcs12 -export -out client.p12 -inkey client.key -in client.crt -certfile ca.crt

```

# 对称加密和非对称加密

## 对称加密

加密和解密都是用的同一个密钥，这个密钥就是对称加密的密钥。

表现形式就是:

client : content  --> (key)  --> encrypted content
server : encrypted content  --> (key)  --> content


## 非对称加密:

加密和解密使用不同的密钥，一个是公钥，一个是私钥。
一般来说,公钥用来加密,私钥用来解密。

表现形式就是:

client  : content  --> (public key)  --> encrypted content
server : encrypted content  --> (private key)  --> content


## 为什么使用非对称加密

只要涉及到密钥的传输，就会有密钥泄露的风险，而非对称加密就是为了解决这个问题的。
对称加密的密钥是需要传输的，而非对称加密的公钥是可以公开的，私钥是不会传输的，这样就解决了密钥传输的问题。

然后这会出现一个问题，就是如何保证公钥的真实性，这就需要数字签名和证书了。

# 证书和数字签名

因为要在网络上传输公钥，所以需要保证公钥的真实性，这就需要证书和数字签名。
证书就是一个包含了公钥和一些其他信息的文件，数字签名就是用来保证证书的真实性的。
因为非对称加密也要传输公钥,避免中间人攻击(大概原理就是:坏人A假装是服务器,把公钥发给客户端,那么客户端就会把信息发送给坏人A),所以公钥都是要通过数字证书认证的.

也就是上面生成证书的这一步:

```bash
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -days 365 -sha256
```

这个ca证书由大家公认的证书机构颁发，这个证书机构就是CA(Certificate Authority)。


也就是客户端在收到服务器的公钥后，会验证这个公钥是否是`CA`颁发的，如果是的话，就可以信任这个公钥。

## 证书和公钥的关系

需要注意的是，证书是包含了公钥的，但是证书不等于公钥，证书是包含了公钥的，还有一些其他信息的文件，比如证书的颁发者，证书的有效期等等。
也就是说证书是包含公钥的.

### X.509

因为证书不仅包含公钥,还包含其他信息,所以就必须有一个标准来定义证书的格式.`X.509`就是这样一个标准。
在上面生成证书的时候，使用的是`x509`的标准，这个标准是用来定义证书的格式的。

简单说,`X.509`就是一个证书的标准.定义了证书的数据结构.比如第几个字节代表什么,这个字段是什么等等.



# 什么是双向认证,什么是单向认证

## 单向认证

就是客户端值验证服务器的证书，服务器不验证客户端的证书。

`https`就是单向认证。

## 双向认证

服务端验证客户端的证书，客户端验证服务端的证书。

这种情况下，客户端也需要有证书，这个证书是由`CA`颁发的，这个证书就是上面生成的`client.crt`。


# 非对称加密和TLS双向认证

TLS是基于非对称加密的，TLS的握手过程就是用来协商对称加密的密钥的。
过程如下(只展示证书参与的过程):

clien --connect--> server : connect
server --server crt --> client : 服务端向客户端出示证书,客户端使用ca证书验证服务端证书
client --client crt --> server : 客户端向服务端出示证书,服务端使用ca证书验证客户端证书

# 常用文件格式解释:

在生成证书的过程,网上可能会有很多文件格式.这里回忆下每一步生成的过程.以自签名证书为例:

1. 生成ca私钥--> ca.key
2. 生成自签名根证书 --> ca.crt
3. 生成服务器私钥 --> server.key
4. 生成证书签名请求(CSR) --> server.csr
5. 使用ca证书签名服务器证书 --> server.crt
.... 省略客户端密钥和公钥生成的过程


## key,pem,der,p12,PCSK#7 ......

网上不同的文章可能生成各种文件.也可能描述的不一样.这里简单解释下.

### pem

`Privacy Enhanced Mail，` 一般为文本格式，
以 -----BEGIN... 开头，以 -----END... 结尾。中间的内容是 BASE64 编码。
这种格式即可以保存私钥也可以保存证书，
有时我们也把PEM 格式的私钥的后缀改为 .key 以区别证书与私钥。具体你可以看文件的内容。

pem强调的是文件的格式,而不是文件的内容.

比如.crt,.csr,.key都是pem格式的文件.

### der,crt,cer

都是基于`X.509`标准的证书格式,只是后缀和格式不一样.

der是二进制格式,而crt和cer是文本格式.

只包含**一个**证书


### PKCS#7,PKCS#12

#### 和pem区别

PKCS说的是一种数据结构,是以二进制编码格式保存的.
而pem是一种编码格式.两者可以互相转换.


#### PKCS#7

保存证书的另一种格式,这种格式可以保存**多个**证书,比如证书链.


#### PKCS#12
包含私钥和证书的格式,这种格式一般用来保存客户端的私钥和证书,这样方便客户端使用.一般有密码保护
一般以.p12结尾.

## csr

Certificate Signing Request，证书签名请求.

CSR 包含的内容：
- 公钥：CSR中包含的是你生成的密钥对中的公钥（不包括私钥）。CA会用这个公钥来生成证书。
- 标识信息：CSR还包含请求者的相关信息，如组织名、域名、国家等。这些信息最终会出现在证书中。
- 签名：CSR文件由请求者使用自己的私钥对请求进行签名，保证请求的真实性和公钥的合法性。

csr只有经过ca签名后才能生成crt证书.



## key

私钥,私钥和公钥是成对的,私钥保存在key中,而公钥保存在crt或者csr中.

## crt

Certificate 的简称，有可能是 PEM 编码格式，也有可能是 DER 编码格式
证书,里面包含了公钥和其他信息,比如证书的颁发者,证书的有效期等等.




# 总结

之前看生成证书的过程,会有生成很多文件,而且不同文章生成的文件后缀不一样,让我一直不理解.

其实生成自签名证书的流程很简单:

1. 生成ca私钥(ca.key)
2. 使用ca私钥生成ca证书(ca.crt)
3. 生成服务器私钥(server.key)
3. 生成服务器证书签名请求(server.csr)
4. 使用ca证书签名服务器证书(server.crt)
5. 生成客户端私钥(client.key)
6. 生成客户端证书签名请求(client.csr)
7. 使用ca证书签名客户端证书(client.crt)
8. (可选)将客户端的私钥和证书生成客户端p12,这一步是方便将客户端的私钥和证书发送给客户端使用(client.p12)


各个文件的作用:

ca.key: 1:用来生成ca证书. 2:结合ca.crt 用来签名服务器和客户端证书
car.crt: 1:结合ca.key用来签名服务器和客户端证书 2: 在服务端和客户端交互的过程中,判断对方发过来的证书是否由自己签名生成的
server.key:  1.解密客户端用server.crt加密的信息
server.crt: 发送给客户端,用来加密信息,客户端使用ca.crt验证
client.key: 1.解密服务端用client.crt加密的信息
clien.crt: 发送给服务端,用来加密信息,服务端使用ca.crt验证


p12: 里面包含了客户端的私钥和证书,客户端拿到p12文件后,将里面的内容解析成,client.key和client.crt.

其中证书可能有不同的格式.但是基本上都是X.509标准的.
所以网上可能会有生成,.pem,.der,.p12文件.其实原理大同小异.
只要区分好私钥,证书就理解其中原理.

