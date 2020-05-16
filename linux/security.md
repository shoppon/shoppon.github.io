# 安全相关

## 证书

**证书校验：**`openssl verify -CAfile cacert cert1 cert2`

**生成密钥：**`openssl genrsa -out RSA.pem`

**密钥格式转换：**`openssl pkcs8 -topk8 -in key_encrypt.pem -passin pass:123456 -out key_pkcs8.pem -nocrypt`

**私钥加密：**`openssl rsa -in RSA.pem -des3 -passout pass:123456 -out E_RSA.pem`

**私钥解密：**`openssl rsa -in E_RSA.pem -passin pass:123456 -out P_RSA.pem`

**查看密钥参数**：`openssl rsa -in key_encrypt.pem -des -passin pass:123456 -text -noout`
