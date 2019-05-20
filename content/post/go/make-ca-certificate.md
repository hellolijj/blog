---
title: "Make Ca Certificate"
date: 2019-05-18T15:42:53+08:00
draft: true
---

参考文档：https://studygolang.com/articles/9959

```bash
# 首先生成私钥
openssl genrsa -out key.pem 2048

# 然后根据私钥提取公钥
openssl rsa -in key.pem -pubout -out key.pub

# 开始生成X509格式的自签名证书,会要求输入区别名DN的各项信息（国家，城市，组织，姓名，email等.
openssl req -x509 -new -days 365 -key key.pem -out cert.crt
```