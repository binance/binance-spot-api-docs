# Simple Binary Encoding (SBE) FAQ

The goal of this document is to explain:

* How to receive SBE responses in the SPOT API.
* How to decode SBE responses.

SBE is a serialization format used for low-latency. 

This implementation is based on the FIX SBE specification.
* [GitHub repository](https://github.com/FIXTradingCommunity/fix-simple-binary-encoding)
* [HTML document](https://www.fixtrading.org/standards/sbe-online)

## How to get an SBE response

* The `Accept` header must include `application/sbe`.
* Provide the schema ID and version in the `X-MBX-SBE` header as `<ID>:<VERSION>`.

Sample request:

```
curl -sX GET -H "Accept: application/sbe" -H "X-MBX-SBE: 1:0" 'https://api.binance.com/api/v3/exchangeInfo?symbol=BTCUSDT'
```

**Notes:**

* If you provide only `application/sbe` in the Accept header:
	* If SBE is not enabled in the exchange, you will receive an HTTP **406 Not Acceptable**.
	* If the `<ID>:<VERSION>` provided in the `X-MBX-SBE` header is malformed or invalid, the response will be an SBE-encoded error.
	* If the `X-MBX-SBE` header is missing, the response will be an SBE-encoded error.
* If you provide both `application/sbe` and `application/json` in the Accept header:
	* If SBE is not enabled in the exchange, the response will fall back to JSON.
	* If the `<ID>:<VERSION>` provided in the `X-MBX-SBE` header is malformed or invalid, the response will fall back to JSON.
	* If the `X-MBX-SBE` header is missing, the response will fall back to JSON.

## Supported APIs

Currently only REST API for SPOT supports SBE.

## SBE Schema

* The schema to use both for the live exchange and SPOT Testnet will be saved in this repository [here](https://github.com/binance/binance-spot-api-docs/tree/master/sbe/schemas). 
* Any updates to the schema will be noted in the [CHANGELOG](../CHANGELOG.md). 

**Regarding Legacy support:**

* SBE schemas are versioned via two XML attributes, `id` and `version`.
	* `id` is incremented when a breaking change is introduced. When this occurs, `version` is reset to 0.
	* `version` is incremented when a non-breaking change is introduced. When this occurs, `id` is not modified.
* When a new schema is live the old schema becomes deprecated. **Deprecation occurs even when the new schema only introduces non-breaking changes.**
* A deprecated schema will be supported **for at least 6 months after deprecation**. <br>For example given this hypothetical timeline:
	* January 3024: Schema id 1 version 0 is released. This is the first version so this is usable once SBE is enabled in the exchange. 
	* March 3024: Schema id 1 version 1 is released. This schema introduces a non-breaking change. 
		* Schema id 1 version 0 is deprecated, but can still be used for at least another 6 months.
	* August 3024: Schema id 2 version 0 is released. This schema introduces a breaking change.
		* Schema id 1 version 0 is deprecated, but can still be used for at least another 1 month.
		* Schema id 1 version 1 is deprecated, but can still be used for at least another 6 months.
    * September 3024: 6 months have passed since the release of Schema id 1 version 1. 
        * Schema id 1 version 0 is retired.
	* February 3025: Schema id 2 version 1 is released. This schema introduces a non-breaking change.
		* Schema id 1 version 1 is retired.
		* Schema id 2 version 0 is deprecated, but can still be used for at least another 6 months.
* HTTP responses will contain a `X-MBX-SBE-DEPRECATED` header for requests specifying a deprecated `<ID>:<VERSION>` in their `X-MBX-SBE` header.
* Requests specifying a retired `<ID>:<VERSION>` will fail with HTTP 400.
* JSON file regarding the schema life-cycle with the dates of the latest, deprecated, and retired schemas for both the live exchange and SPOT Testnet will be saved in this repository [here](https://github.com/binance/binance-spot-api-docs/tree/master/sbe/schemas). <br> Below is an example JSON based on the hypothetical timeline above:

```json
{
    "environment": "PROD",
    "latestSchema": {
        "id": 2,
        "version": 1,
        "releaseDate": "3025-02-01" 
    },
    "deprecatedSchemas": [
        {
            "id": 2,
            "version": 0,
            "releaseDate": "3024-08-01",
            "deprecatedDate": "3025-02-01" 
        }
    ],
    "retiredSchemas": [
        {
            "id": 1,
            "version": 1,
            "releaseDate": "3024-03-01",
            "deprecatedDate": "3024-08-01", 
            "retiredDate": "3025-02-01",
        },
        {
            "id": 1,
            "version": 0,
            "releaseDate": "3024-01-01",
            "deprecatedDate": "3024-03-01",
            "retiredDate": "3024-09-01",
        }
    ]
}
```

## Generate SBE decoders:

1. Download the schema ([`spot_latest.xml`](../sbe/schemas/spot_latest.xml))
2. Clone and build [`simple-binary-encoding`](https://github.com/real-logic/simple-binary-encoding):
```shell
 $ git clone https://github.com/real-logic/simple-binary-encoding.git
 $ cd simple-binary-encoding
 $ ./gradlew
```
3. Run the SbeTool code generator. (Here are samples for [Java](https://github.com/binance/binance-sbe-java-sample-app), [C++](https://github.com/binance/binance-sbe-cpp-sample-app) and [Rust](https://github.com/binance/binance-sbe-rust-sample-app) decoding the payload from Exchange Information.)

### Decimal field encoding

Unlike the FIX SBE specification, decimal fields have their mantissa and exponent fields encoded separately as primitive fields in order to minimize payload size and the number of encoded fields within messages.

### Timestamp field encoding

Timestamps in SBE responses are in microseconds. This differs from JSON responses, which contain millisecond timestamps.

### Custom field attributes in the schema file

A few field attributes prefixed with `mbx:` were added to the schema file for documentation purposes:
- `mbx:exponent`: Points to the exponent field corresponding to the mantissa field
- `mbx:jsonPath`: Contains the name of the equivalent field in the JSON response
- `mbx:jsonValue`: Contains the name of the equivalent ENUM value in the JSON response






