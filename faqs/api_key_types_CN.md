# API Key 类型

币安 API 需要 API Key 才能访问经过身份验证的接口以进行交易，账户历史记录等。

我们支持多种类型的 API key：

- Ed25519（推荐）
- HMAC
- RSA

本文档概述了受支持的 API Keys。

**我们建议使用 Ed25519 API keys**，因为它在所有受支持的 API key 类型中提供最佳性能和安全性。

请读 [REST API](../rest-api_CN.md#需要签名的接口-trade-与-user_data) 或者 [WebSocket API](../web-socket-api_CN.md#请求鉴权类型) 文档以了解如何使用不同的 API Key 类型。

## Ed25519 

Ed25519 keys 使用非对称加密技术。
您只与币安共享您的 public key 并在本地使用 private key 签署 API 请求。
币安 API 会使用 public key 来验证您的请求签名。

Ed25519 Keys 提供与 3072 bits 的 RSA keys 相当的安全性，但是 key 更小，签名更小，签名的计算更快。

**我们建议使用 Ed25519 API keys**

Ed25519 key 例子:

```
-----BEGIN PUBLIC KEY-----
MCowBQYDK2VwAyEAgmDRTtj2FA+wzJUIlAL9ly1eovjLBu7uXUFR+jFULmg=
-----END PUBLIC KEY-----
```

Ed25519 签名例子:

```
E7luAubOlcRxL10iQszvNCff+xJjwJrfajEHj1hOncmsgaSB4NE+A/BbQhCWwit/usNJ32/LeTwDYPoA7Qz4BA==
```

## HMAC

HMAC keys 使用对称加密技术。
币安生成并与您共享一个 secret key，您可以使用该 secret key 对 API 请求进行签名。
币安 API 使用相同的共享 secret key 来验证您的请求签名。

HMAC 签名可以快速计算和压缩。<br>
但是，由于共享 secret key 必须在多方之间共享，这就不如 Ed25519 或 RSA keys 使用的非对称加密技术那么安全。

**不建议使用 HMAC keys。** 我们建议换成并使用非对称 API Keys，例如 Ed25519 或 RSA。

HMAC key 例子:

```
Fhs4lGae2qAi6VNjbJjebUAwXrIChb7mlf372UOICMwdKaNdNBGKtfdeUff2TTTT
```

HMAC 签名例子:

```
7f3fc79c57d7a70d2b644ad4589672f4a5d55a62af2a336a0af7d4896f8d48b8
```

## RSA

RSA keys 使用非对称加密技术。 <br>
您只与币安共享您的 public key 并在本地使用 private key 签署 API 请求。
币安 API 会使用 public key 来验证您的请求签名。

我们支持 2048 和 4096 bits 的 RSA keys。

虽然 RSA keys 比 HMAC keys 更安全，RSA 签名比 HMAC 和 Ed25519 大很多，这会降低性能。

RSA (2048 bits) 例子:

```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyfKiFXpcOhF5rX1XxePN
akwN7Etwtn3v05cZNY+ftDHbVZHs/kY6Ruj5lhxVFAq5dv7Ba9/4jPijXuMuIc6Y
8nUlqtrrxC8DEOAczw9SKATDYZN9nbLfYlbBFfHzRQUXdAtYCPI6XtxmJBS7aOBb
4nZe1SVm+bhLrp0YQnx2P0s+37qkGeVn09m6w9MnWxjgCkkYFPWQkXIu5qOnwx6p
NfqDmFD7d7dUc/6PZQ1bKFALu/UETsobmBk82ShbrBhlc0JXuhf9qBR7QASjHjFQ
2N+VF2PfH8dm5prZIpz/MFKPkBW4Yuss0OXiD+jQt1J2JUKspLqsIqoXjHQQGjL7
3wIDAQAB
-----END PUBLIC KEY-----
```

RSA (2048 bits) 签名例子::

```
wS6q6h77AvH1TqwInoTDdWIIubRCiUP4RLG++GI24twL3BMtX0EEV+YT1eH8Hb8bLe0Rb9OhOHbt1CC3aurzoCTgZvhNek47mg+Bpu8fwQ7eRkXEiWBx5C8BNN73JwnnkZw4UzYvqiwAs162jToV8AL0eN043KJ3MEKCy3C6nyeYOFSg+1Cp637KtAZk3z7aHknSu7/PXSPuwMIpBgFctf8YKGZFAVRbgwlcgUDhXyaGts6OFePGy0jkZKJHawb/w5hoatatsfVmVC4hZ8fsfystQ9k5DNjTm7ROApWaXy9BsfAYcj13O424mqlpkKG4EGnIjOIWB/pRDDQEm2O/xg==
```
