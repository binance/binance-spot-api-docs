# API Key Types

Binance APIs require an API key to access authenticated endpoints for trading, account history, etc.

We support several types of API keys:

- Ed25519 (recommended)
- HMAC
- RSA

This document provides an overview of supported API keys.

**We recommend to use Ed25519 API keys** as it should provide the best performance and security out of all supported key types.

Read [REST API](../rest-api.md#signed-trade-and-user_data-endpoint-security) or [WebSocket API](../web-socket-api.md#request-security) documentation to learn how to use different API keys.

## Ed25519 

Ed25519 keys use asymmetric cryptography.
You share your public key with Binance and use the private key to sign API requests.
Binance API uses the public key to verify your signature.

Ed25519 keys provide security comparable to 3072-bit RSA keys, but with considerably smaller key, smaller signature size, and faster signature computation.

**We recommend to use Ed25519 API keys.**

Sample Ed25519 key:
```
-----BEGIN PUBLIC KEY-----
MCowBQYDK2VwAyEAgmDRTtj2FA+wzJUIlAL9ly1eovjLBu7uXUFR+jFULmg=
-----END PUBLIC KEY-----
```

Sample Ed25519 signature:

```
E7luAubOlcRxL10iQszvNCff+xJjwJrfajEHj1hOncmsgaSB4NE+A/BbQhCWwit/usNJ32/LeTwDYPoA7Qz4BA==
```

## HMAC

HMAC keys use symmetric cryptography.
Binance generates and shares with you a secret key which you use to sign API requests.
Binance API uses the same shared secret key to verify your signature.

HMAC signatures are quick to compute and compact. <br>
However, the shared secret must be shared between multiple parties which is less secure than asymmetric cryptography used by Ed25519 or RSA keys.

**HMAC keys are deprecated.** We recommend to migrate to asymmetric API keys, such as Ed25519 or RSA.

Sample HMAC key:

```
Fhs4lGae2qAi6VNjbJjebUAwXrIChb7mlf372UOICMwdKaNdNBGKtfdeUff2TTTT
```

Sample HMAC signature:

```
7f3fc79c57d7a70d2b644ad4589672f4a5d55a62af2a336a0af7d4896f8d48b8
```

## RSA

RSA keys use asymmetric cryptography. <br>
You share your public key with Binance and use the private key to sign API requests. <br>
Binance API uses the public key to verify your signature.

We support 2048 and 4096 bit RSA keys.

While RSA keys are more secure than HMAC keys,
RSA signatures are much larger than HMAC and Ed25519 which can lead to a degradation to performance.

Sample RSA key (2048 bits):

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

Sample RSA signature (2048 bits):

```
wS6q6h77AvH1TqwInoTDdWIIubRCiUP4RLG++GI24twL3BMtX0EEV+YT1eH8Hb8bLe0Rb9OhOHbt1CC3aurzoCTgZvhNek47mg+Bpu8fwQ7eRkXEiWBx5C8BNN73JwnnkZw4UzYvqiwAs162jToV8AL0eN043KJ3MEKCy3C6nyeYOFSg+1Cp637KtAZk3z7aHknSu7/PXSPuwMIpBgFctf8YKGZFAVRbgwlcgUDhXyaGts6OFePGy0jkZKJHawb/w5hoatatsfVmVC4hZ8fsfystQ9k5DNjTm7ROApWaXy9BsfAYcj13O424mqlpkKG4EGnIjOIWB/pRDDQEm2O/xg==
```
