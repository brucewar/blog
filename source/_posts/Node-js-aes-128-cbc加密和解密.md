---
title: Node.js aes-128-cbc加密和解密
date: 2017-05-06 11:31:52
categories: Node.js
tags: [AES,aes-128-cbc]
---

Java程序中经常使用的AES加密模式是**AES/CBC/PKCS5Padding**，在Node.js中对应的是**aes-128-cbc**加密算法。

为此，我们需要引入Node.js的*crypto*模块，详细说明请查看[官方文档](https://nodejs.org/dist/latest-v6.x/docs/api/crypto.html)。

```javascript
'use strict';
const crypto = require('crypto');
const ALG_STRING = 'aes-128-cbc',
  KEY = 'wjl891014#gmail.com',
  IV = [ 0xcb, 0x53, 0x03, 0x0f, 0xe0, 0x79, 0x9d, 0xdc, 0x80, 0xa9, 0x83, 0xf1, 0x03, 0xb6, 0x59, 0x83 ];

const md5sum = str => {
  return crypto.createHash('md5')
    .update(str)
    .digest()
    .slice(0, 16);
};

const encrypt = str => {
  const key = md5sum(KEY);
  // key和IV 必须是16位或32位
  const cipher = crypto.createCipheriv(ALG_STRING, key, Buffer.from(IV));
  cipher.setAutoPadding(true);
  const cipherChunks = [];
  cipherChunks.push(cipher.update(str, 'utf8', 'binary'));
  cipherChunks.push(cipher.final('binary'));
  return Buffer.from(cipherChunks.join(''), 'binary').toString('base64');
};

const decrypt = str => {
  const key = md5sum(KEY);
  const decipher = crypto.createDecipheriv(ALG_STRING, key, Buffer.from(IV));
  decipher.setAutoPadding(true);
  const cipherChunks = [];
  cipherChunks.push(decipher.update(Buffer.from(str, 'base64').toString('binary'), 'binary', 'utf8'));
  cipherChunks.push(decipher.final('utf8'));
  return cipherChunks.join('');
};

const data = 'bd2983b21dd2aeb1e1453ab0273b4dc';
console.log(`加密前：${data}`);
const encryptData = encrypt(data);
console.log(`加密后：${encryptData}`);
const decryptData = decrypt(encryptData);
console.log(`解密后：${decryptData}`);
```
