# Asymmetric Key Generator

This simple tool can be used to generate an Ed25519 PKCS#8 and RSA key pairs (private and public key).

There's two methods to run the tool, you can either download or build from source code.

## Download
Prebuild apps can be found in [Releases](https://github.com/binance/asymmetric-key-generator/releases).

To verify the package's integrity, download both the app and the checksum file to the same directory:

```shell
-rw-r--r--@  1 john  staff   156M 18 Nov 17:02 AsymmetricKeyGenerator-0.5.0-universal.dmg
-rw-r--r--@  1 john  staff   102B 18 Nov 19:01 AsymmetricKeyGenerator-0.5.0-universal.dmg.CHECKSUM
```

Then run sha256 checksum:

```shell
sha256sum -c AsymmetricKeyGenerator-0.5.0-universal.dmg.CHECKSUM
```

If it passes the integrity check, it'll return `AsymmetricKeyGenerator-0.5.0-universal.dmg: OK`

## Build from source code

Obtain the source code locally and go through the following steps:

```javascript

// install package
npm install

// run locally
npm run start

// build packages locally into the "dist" folder.
npm run dist

```

## Usage

1. Open the app;

2. Choose the key type; Recommend to keep the default value (`Ed25519`), then click the button `Generate Key Pair`;

3. Below on the left column is the `Private Key`, which should be stored in a secure location on your local disk (by using the `Save` button) and must never be shared with anyone;

4. The right column is the `Public Key`. Click `Save` button to save into local disk. This key can be shared with others to verify the `Private Key` signed data;

## Contribution
Contributions are welcome!
If you've found a bug within this project, please open an issue to discuss what you would like to change.

## License
MIT
