---
layout: article
title: 多语言 AES 加解密
tags: []
key: 07cc2631-c829-42b6-ba44-d40279934ee2
---

主要是 AES-CBC、AES-ECB 加解密.

<!--more-->

## JavaScript 实现

### 依赖包

```js
npm install crypto-js
```

### Base64

```js
const CryptoJS = require('crypto-js');
  
function Base64Encrypt(word) {
    var str = CryptoJS.enc.Utf8.parse(word);
    var base64 = CryptoJS.enc.Base64.stringify(str);
    return base64
}

function Base64Decrypt(word) {
    let words = CryptoJS.enc.Base64.parse(word);
    let str = words.toString(CryptoJS.enc.Utf8);
    return str
}

console.log(Base64Encrypt('张'));
console.log(Base64Decrypt('5byg'));
```

### AES-CBC

```js
const CryptoJS = require('crypto-js');
const key = CryptoJS.enc.Utf8.parse('1234567890123456');  //十六位十六进制数作为密钥
const iv = CryptoJS.enc.Utf8.parse('1234567890123456');   //十六位十六进制数作为密钥偏移量

//解密方法 参数时 BASE64 编码密文
function AesCbcDecrypt(word) {
    let decrypt = CryptoJS.AES.decrypt(word, key, { iv: iv, mode: CryptoJS.mode.CBC, padding: CryptoJS.pad.Pkcs7 });
    let decryptedStr = decrypt.toString(CryptoJS.enc.Utf8);
    return decryptedStr.toString();
}

//加密方法  返回 BASE64 编码密文
function AesCbcEncrypt(word) {
    let srcs = CryptoJS.enc.Utf8.parse(word);
    let encrypted = CryptoJS.AES.encrypt(srcs, key, { iv: iv, mode: CryptoJS.mode.CBC, padding: CryptoJS.pad.Pkcs7 });
    return encrypted.toString();
}

console.log(AesCbcEncrypt('123456'));
console.log(AesCbcDecrypt('1jdzWuniG6UMtoa3T6uNLA=='));
```

### AES-ECB

```js
const CryptoJS = require('crypto-js');
const key = CryptoJS.enc.Utf8.parse('1234567890123456');  //十六位十六进制数作为密钥
  
//解密方法
function AesEcbDecrypt(word) {
    let decrypt = CryptoJS.AES.decrypt(word, key, { mode: CryptoJS.mode.ECB, padding: CryptoJS.pad.Pkcs7 });
    let decryptedStr = decrypt.toString(CryptoJS.enc.Utf8);
    return decryptedStr.toString();
}

//加密方法  返回 BASE64
function AesEcbEncrypt(word) {
    let srcs = CryptoJS.enc.Utf8.parse(word);
    let encrypted = CryptoJS.AES.encrypt(srcs, key, { mode: CryptoJS.mode.ECB, padding: CryptoJS.pad.Pkcs7 });
    return encrypted.toString();
}


// AES-ECB
console.log(AesEcbEncrypt('123456'));
console.log(AesEcbDecrypt('yXVUkR45PFz0UfpbDB8/ew=='));
```

## Java 实现

### 依赖包

* JDK 8
* apache 的开源 commons-codec 包.

### Base64

```java
public static void main(String[] args) throws Exception {
    System.out.println(Base64.encodeBase64String("张".getBytes("UTF-8")));
    System.out.println(new String(Base64.decodeBase64("5byg".getBytes("UTF-8"))));
}
```

### AES-CBC

```java
private static String ALG = "AES";
private static String ALGORITHM = "AES/CBC/PKCS5Padding";//算法/模式/补码方式
private static String ENCODING = "utf-8"; //编码


/**
 * 加密
 *
 * @param text   明文
 * @param secret 秘钥
 * @param iv     偏移向量
 * @return 密文,使用 Base64 编码
 */
public static String encrypt(String text, String secret, String iv) {
    try {
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        byte[] raw = secret.getBytes();
        SecretKey secretKey = new SecretKeySpec(raw, ALG);
        IvParameterSpec ivParameterSpec = new IvParameterSpec(iv.getBytes());//使用CBC模式,需要一个向量iv,可增加加密算法的强度
        cipher.init(Cipher.ENCRYPT_MODE, secretKey, ivParameterSpec);
        byte[] encrypted = cipher.doFinal(text.getBytes(ENCODING));
        return Base64.encodeBase64String(encrypted);//此处使用BASE64做转码.
    } catch (Exception e) {
        return null;
    }
}

/**
 * 解密
 *
 * @param text   密文,使用 Base64 编码
 * @param secret 秘钥
 * @param iv     偏移向量
 * @return 明文
 */
public static String decrypt(String text, String secret, String iv) {
    try {
        byte[] raw = secret.getBytes("ASCII");
        SecretKey secretKey = new SecretKeySpec(raw, ALG);
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        IvParameterSpec ivParameterSpec = new IvParameterSpec(iv.getBytes());
        cipher.init(Cipher.DECRYPT_MODE, secretKey, ivParameterSpec);
        byte[] encrypted1 = Base64.decodeBase64(text);//先用base64解密
        byte[] original = cipher.doFinal(encrypted1);
        return new String(original, ENCODING);
    } catch (Exception ex) {
        return null;
    }
}
  
public static void main(String[] args) throws Exception {
    String SECRET = "1234567890123456";// 秘钥
    String IV = "1234567890123456";// 偏移向量

    String text = "123456";// 需要加密的字串
    System.out.println("加密前的字串是:" + text);
    String enString = encrypt(text, SECRET, IV);// 加密
    System.out.println("加密后的字串是:" + enString);
    System.out.println("1jdzWuniG6UMtoa3T6uNLA==".equals(enString));
    String DeString = decrypt(enString, SECRET, IV);// 解密
    System.out.println("解密后的字串是:" + DeString);
}
```

### AES-ECB

```java
private static String ALG = "AES";
private static String ENCODING = "utf-8"; //编码
private static String ALGORITHM = "AES/ECB/PKCS5Padding"; // 算法/模式/补码方式

/**
 * 加密
 *
 * @param text   明文
 * @param secret 秘钥
 * @return 密文,使用 Base64 编码
 */
public static String encrypt(String text, String secret) {
    try {
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        byte[] raw = secret.getBytes();
        SecretKey secretKey = new SecretKeySpec(raw, ALG);
        cipher.init(Cipher.ENCRYPT_MODE, secretKey);
        byte[] encrypted = cipher.doFinal(text.getBytes(ENCODING));
        return Base64.encodeBase64String(encrypted);//此处使用BASE64做转码.
    } catch (Exception e) {
        return null;
    }
}

/**
 * 解密
 *
 * @param text   密文,使用 Base64 编码
 * @param secret 秘钥
 * @return 明文
 */
public static String decrypt(String text, String secret) {
    try {
        Cipher cipher = Cipher.getInstance(ALGORITHM);
        byte[] raw = secret.getBytes();
        SecretKey secretKey = new SecretKeySpec(raw, ALG);
        cipher.init(Cipher.DECRYPT_MODE, secretKey);
        byte[] hexBytes = Base64.decodeBase64(text);//先用base64解密
        byte[] plainBytes = cipher.doFinal(hexBytes);
        return new String(plainBytes, ENCODING);
    } catch (Exception e) {
        return null;
    }
}

public static void main(String[] args) {
    String SECRET = "1234567890123456";// 秘钥
    String text = "123456";// 需要加密的字串
    System.out.println("加密前的字串是:" + text);
    String enString = encrypt(text, SECRET);// 加密
    System.out.println("加密后的字串是:" + enString);
    System.out.println("yXVUkR45PFz0UfpbDB8/ew==".equals(enString));
    String DeString = decrypt(enString, SECRET);// 解密
    System.out.println("解密后的字串是:" + DeString);
}
```

## Python 实现

### 依赖包

* Python 3
* pycryptodome

### AES-CBC

```python
# -*- coding: utf-8 -*-

import base64
from Crypto.Cipher import AES


def str_to_bytes(s):
    '''
    字符串转 byte 数组, utf8 编码
    '''
    return s.encode('utf8')

def bytes_to_str(bs):
    '''
    byte 数组转字符串, utf8 编码
    '''
    return bs.decode('utf8')

def pad(s):
    '''
    填充数据
    '''
    bs = AES.block_size
    return s + (bs - len(s) % bs) * chr(bs - len(s) % bs)

def unpad(s):
    '''
    去掉填充数据
    '''
    return s[0:-ord(s[-1])]

# 十六位十六进制数作为密钥
KEY = '1234567890123456'
# 六位十六进制数作为密钥偏移量
IV = '1234567890123456'

# 是否调试
is_debug = True

def log(s):
    if is_debug:
        print(s)


def encrypt_aes_cbc(text, key, iv):
    '''
    AES/CBC/PKCS5Padding  加密
    '''
    # 初始化加密器
    aes = AES.new(str_to_bytes(key), AES.MODE_CBC, iv=str_to_bytes(iv))
    # 填充长度
    text_pad = pad(text)
    # 字符串转 byte 数组
    text_bytes = str_to_bytes(text_pad)
    # 加密获取新的 byte 数组
    encrypt_bytes = aes.encrypt(text_bytes)
    # byte 数组转 base64 编码 byte 数组
    encrypt_base64 = base64.encodebytes(encrypt_bytes)
    # bytes 数组转字符串
    encrypted_text = bytes_to_str(encrypt_base64)
    return encrypted_text.replace('\n', '')


def decrypt_aes_cbc(text, key, iv):
    '''
    AES/CBC/PKCS5Padding  解密
    '''
    # 初始化加密器
    aes = AES.new(str_to_bytes(key), AES.MODE_CBC, iv=str_to_bytes(iv))
    # 字符串转 byte 数组
    text_base64 = str_to_bytes(text)
    # byte 数组 base64 反向编码
    text_bytes = base64.decodebytes(text_base64)
    # 解密获取原始 byte 数组
    decrypt_bytes = aes.decrypt(text_bytes)
    # byte 数组转字符串
    decrypt_text = bytes_to_str(decrypt_bytes)
    decrypted_text = unpad(decrypt_text)
    return decrypted_text

if __name__ == '__main__':
    print(encrypt_aes_cbc('123456', KEY, IV))
    print(decrypt_aes_cbc('1jdzWuniG6UMtoa3T6uNLA==', KEY, IV))
```

### AES-ECB

```python
# -*- coding: utf-8 -*-

import base64
from Crypto.Cipher import AES

def str_to_bytes(s):
    '''
    字符串转 byte 数组, utf8 编码
    '''
    return s.encode('utf8')

def bytes_to_str(bs):
    '''
    byte 数组转字符串, utf8 编码
    '''
    return bs.decode('utf8')

def pad(s):
    '''
    填充数据
    '''
    bs = AES.block_size
    return s + (bs - len(s) % bs) * chr(bs - len(s) % bs)

def unpad(s):
    '''
    去掉填充数据
    '''
    return s[0:-ord(s[-1])]

def encrypt_aes_cbc(text, key):
    # 字符串转 byte 数组
    text_bytes = str_to_bytes(pad(text))
    # 初始化加密器
    aes = AES.new(str_to_bytes(key), AES.MODE_ECB)  
    # 加密
    encrypted_text = bytes_to_str(base64.encodebytes(aes.encrypt(text_bytes))).replace('\n', '')
    return encrypted_text

def decrypt_aes_cbc(text, key):
    # 字符串转 byte 数组
    text_base64 = str_to_bytes(pad(text))
    # base64 byte 数组 转 byte 数组
    text_bytes = base64.decodebytes(text_base64)
    # 初始化加密器
    aes = AES.new(str_to_bytes(key), AES.MODE_ECB)  
    # 解密
    decrypted_text = unpad(bytes_to_str(aes.decrypt(text_bytes)))
    return decrypted_text

if __name__ == '__main__':
    KEY = '1234567890123456'  # 密码
    print(encrypt_aes_cbc('123456', KEY))
    print(decrypt_aes_cbc('yXVUkR45PFz0UfpbDB8/ew==', KEY))
```
