---
title: PHP中aes加密解密实现
date: 2018-03-30 00:50:00
categories:
- PHP
tags: 
- PHP
- aes
---

常用的加密分为对称加密和非对称加密，对称加密就是发送方和接收方都用同一个秘钥进行加密解密，而非对称加密则使用一对公钥和私钥来进行加密，发送发只需要用接收方的公钥将数据加密即可。

## AES

AES是一种常见的对称加密算法，英语：Advanced Encryption Standard，又称Rijndael加密，它是一种分块加密方法，换句话说就是将明文块分成一组组小部分然后进行加密再组合，而根据分组大小可分为：AES-128,AES-196,AES-256三种，对应的分组大小分别是128bit，196bit，256bit。

而如何将这些明文块进行加密组合，又分为四种模式：ECB、CBC、CFB、OFB，除了第一种模式，后面三种模式都需要一个初始向量iv来辅助加密。初始向量的大小也是同区块大小保持一致，是个固定长度的随机字串，初始向量只是为了给加密算法提供一个可用的种子。

还有一点是，如果需要加密的源数据长度不是分组大小的整数倍，那么就需要对数据进行填充，一般的填充算法有PKSC5和PKSC7填充算法。举个例子，如果源数据大小是10bytes，如果进行分组，还需要6bytes才能满足分组的要求。那么只需要在后面补上6个6即可：

```
x x x x x x x x x x 6 6 6 6 6 6
```

如果刚好不用填充，还需要再补齐区块大小个字节，这么做是为了在去除填充数据时，可以准确无误。

## PHP 实现 AES 

随着Mcrypt扩展从 PHP 7.1.0 开始废弃，我们将利用OpenSSL代替Mcrypt实现AES加解密；

```
public function encrypt_decrypt($action, $string) {
        $output = false;
        $encrypt_method = "AES-256-CBC";
        $secret_key = 'This is my secret key';
        $secret_iv = 'This is my secret iv';
        $key = hash('sha256', $secret_key);
        $iv = substr(hash('sha256', $secret_iv), 0, 16);
        if ( $action == 'encrypt' ) {
            $output = openssl_encrypt($string, $encrypt_method, $key, 0, $iv);
            $output = base64_encode($output);
        } else if( $action == 'decrypt' ) {
            $output = openssl_decrypt(base64_decode($string), $encrypt_method, $key, 0, $iv);
        }
        return $output;
    }
```

