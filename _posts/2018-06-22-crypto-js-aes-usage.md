---
layout: default
title: crypto-js AES 使用经验
---


关于 `crypto-js` AES的使用, encrypt 方法的 key 参数, 官方提供的例子 key 是string 类型传入, 与其它语言string 密钥不同, 很容易误导大家.

**官方例子:**

```javascript
// Encrypt
var ciphertext = CryptoJS.AES.encrypt('my message', 'secret key 123');

console.log(ciphertext.toString()); //这里每次得到的结果都是不一样的

// Decrypt
var bytes  = CryptoJS.AES.decrypt(ciphertext.toString(), 'secret key 123');
```

网上有其它例子, 提供了AES的"正确使用":

```javascript
var str = '123456';
var key = '0123456789abcdef'; // 密钥 16 位
var iv = 'abcdef0123456789';  // 初始向量 initial vector 16 位

key = CryptoJS.enc.Utf8.parse(key);
iv = CryptoJS.enc.Utf8.parse(iv);

var options = {
    iv: iv,
    mode: CryptoJS.mode.CBC,
    padding: CryptoJS.pad.Pkcs7
};
 
var encrypted = CryptoJS.AES.encrypt(str, key, options);

encrypted = encrypted.toString(); // 转换为字符串
console.log(encrypted); // 9FTrAdsYkbHrRsZ1A0IsDw==

var decrypted = CryptoJS.AES.decrypt(encrypted, key, options);
decrypted = CryptoJS.enc.Utf8.stringify(decrypted); // 转换为 utf8 字符串
```

对应 PHP 使用:

```php
$str = '123456';
$key = '0123456789abcdef';
$iv = 'abcdef0123456789';
$options = OPENSSL_RAW_DATA | OPENSSL_PKCS1_PADDING;

$encrypted = openssl_encrypt($str, 'aes-128-cbc', $key, $options, $iv);

echo base64_encode($encrypted), "\n"; //9FTrAdsYkbHrRsZ1A0IsDw==

echo openssl_decrypt($encrypted, 'aes-128-cbc',$key , $options, $iv), "\n";
```

那官方提供的例子, key 为 string 类型, 会经过什么样的处理呢?

这里先提供下 `CryptoJS.AES.encrypt` 的参数说明

```javascript
/**
 * 加密消息
 *
 * @param {WordArray|string} message 加密字符串
 * @param {WordArray|string} key     密码
 * @param {Object} cfg               (可选) 加密配置选项.
 *
 * Default cfg:
 *   {
 *      iv: WordArray,               // IV
 *      mode: CryptoJS.mode.CBC,     // mode 支持 CBC,CFB,CTR,ECB,OFB
 *      padding: CryptoJS.pad.Pkcs7, // padding 支持 Pkcs7,AnsiX923,Iso10126, NoPadding,ZeroPadding
 *      kdf: CryptoJS.kdf.OpenSSL,   // EvpKDF
 *   }
 *
 * @return {CipherParams} A cipher params object.
 */
CryptoJS.AES.encrypt(str, key, cfg);
```

如果 key 为 string 类型, 会经过 `CryptoJS.lib.PasswordBasedCipher` 处理, 使用 `cfg.kdf` 生成新的 `WordArray`, 
`cfg.kdf` 默认是 `CryptoJS.kdf.OpenSSL`(即 `EvpKDF`).

**以下是相应代码执行的stack**

- https://github.com/brix/crypto-js/blob/develop/src/cipher-core.js#L176-L178
- https://github.com/brix/crypto-js/blob/develop/src/cipher-core.js#L812


IV设置说明及误区
--------------

- ECB 模式不需要 iv 参数
- 需要用到向量(iv)的模式, 不建议 iv 与 key 一致, 网上好多例子提供的 `key == iv`, 这是不正确的. 
  因为IV并不是保密的, 它的密文（通常是它的第一个块）已暴露了.
  给出密文的攻击者可以恢复前128位, 将它们用作密钥并恢复原始明文.

其它参考:
--------

- [AES key equal to IV (CBC mode)](https://crypto.stackexchange.com/questions/31583/aes-key-equal-to-iv-cbc-mode)
- [JavaScript Crypto-JS 使用手册](https://blog.zhengxianjun.com/2015/05/javascript-crypto-js/)
