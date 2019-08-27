# HTTP API
dYdX offers a few HTTP APIs for retrieving information about the protocol, and submitting orders to
our DEX. Feel free to use these APIs to build your own applications on top of dYdX. Please feel
free to let us know via Intercom or Telegram if you have any questions or experience any issues.

All of these endpoints live at `https://api.dydx.exchange/`

e.g. `https://api.dydx.exchange/v1/orders`

## Orderbook

### Introduction

The following API endpoints allow for submitting and retrieving orders from the dYdX orderbook.
This orderbook is what's frequently referred to as a "Matching Model" orderbook. This means that
all orders are submitted to the blockchain by dYdX itself. You do not need to provide gas fees
or send on-chain transactions yourself. This is especially useful for traders and market makers who
wish to be able to quickly cancel their orders without waiting for a transaction to be mined.

The below documents the underlying HTTP API. For easier implementation we recommend using the official [Python Client](python.md) or [TypeScript Client](typescript.md#api). We may build clients for other languages in the future, so if you have other language/framework needs, please let us know.

In order to submit an order, you (the maker) must first create a JSON object that specifies the
details of your order. Once you create this object you must sign it with your Ethereum private key,
and put the result in the `typedSignature` field. Note: The `typedSignature` is omitted before
signing, and added only after signing the message.

The order data is hashed according to [EIP712](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md).
This includes the exact order format and version as well as information about the verifying contract and the chainId of the network. You can see working examples of signing in the [LimitOrders](https://github.com/dydxprotocol/solo/blob/master/src/modules/LimitOrders.ts) module of Solo.js.

When creating your order you _must_ specify the takerAccountOwner as `0xf809e07870dca762B9536d61A4fBEF1a17178092` and the takerAccountNumber
as `0`, otherwise your order will be rejected.

After this is done, the order is ready to be submitted to the API.

__Order fields__

|Field Name|JSON type|Description|
|----------|---------|-----------|
|makerMarket|string|The Solo [market](https://docs.dydx.exchange/#/overview?id=markets) of the Maker amount|
|takerMarket|string|The Solo [market](https://docs.dydx.exchange/#/overview?id=markets) of the Taker amount|
|makerAmount|string|The amount of token the Maker is offering in base units|
|takerAmount|string|The amount of token the Maker is requesting from the taker base units|
|makerAccountOwner|string|The Ethereum address of the Maker.|
|takerAccountOwner|string|The Ethereum address of the Taker. This must be to the dYdX account owner listed above|
|makerAccountNumber|string|The Solo [account number](https://docs.dydx.exchange/#/overview?id=markets) of the Maker|
|takerAccountNumber|string|The Solo [account number](https://docs.dydx.exchange/#/overview?id=markets) of the Taker. This must be set to teh dYdX account number listed above|
|expiration|string|The time in unix seconds at which this order will be expired and can no longer be filled. Use `"0"` to specify that there is no expiration on the order.|
|salt|string|A random number to make the orderHash unique.|
|typedSignature|string|The signature of the order.|

Example:
```json
{
    "makerMarket": "0",
    "takerMarket": "1",
    "makerAmount": "10000000000",
    "takerAmount": "20000000000",
    "makerAccountOwner": "0x3E5e9111Ae8eB78Fe1CC3bb8915d5D461F3Ef9A9",
    "makerAccountNumber": "111",
    "takerAccountOwner": "0x28a8746e75304c0780E011BEd21C72cD78cd535E",
    "takerAccountNumber": "222",
    "expiration": "4294967295",
    "salt": "100",
    "typedSignature": "0xd9c006cf9066e89c2e75de72604751f63985f173ca3c69b195f1f5f445289a1f2229c0475949858522c821190c5f1ec387f31712bd21f6ac31e4510d5711c2681f00"
  },
};
```

### GET /v1/dex/pairs

Description:
Returns all dex-compatible pairs. Be aware that there are two "pairs" for each unique order pair.

For example, in the unique pair WETH and DAI, there are two pairs, one for each side of the book.
The pairs are always named as `MAKER-TAKER`.

Headers:
```
Content-Type: application/json
```

Example Response Body:
```json
{
    "pairs": [
		{
            "uuid": "e401535b-e43a-4a79-933f-7c1950cabbdf",
            "name": "DAI-WETH",
            "createdAt": "2019-08-13T19:12:27.386Z",
            "updatedAt": "2019-08-13T19:12:27.386Z",
            "deletedAt": null,
            "makerCurrencyUuid": "b656c441-68ab-4776-927c-d894f4d6483b",
            "takerCurrencyUuid": "84298577-6a82-4057-8523-27b05d3f5b8c",
            "makerCurrency": {
                "uuid": "b656c441-68ab-4776-927c-d894f4d6483b",
                "symbol": "DAI",
                "contractAddress": "0x89d24a6b4ccb1b6faa2625fe562bdd9a23260359",
                "decimals": 18,
                "soloMarket": 1,
                "createdAt": "2019-08-13T19:12:27.365Z",
                "updatedAt": "2019-08-13T19:12:27.365Z",
                "deletedAt": null
            },
            "takerCurrency": {
                "uuid": "84298577-6a82-4057-8523-27b05d3f5b8c",
                "symbol": "WETH",
                "contractAddress": "0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2",
                "decimals": 18,
                "soloMarket": 0,
                "createdAt": "2019-08-13T19:12:27.227Z",
                "updatedAt": "2019-08-13T19:12:27.227Z",
                "deletedAt": null
            }
        },
        {
            "uuid": "65354d23-f9a0-49fa-b823-4c0592e0fa60",
            "name": "WETH-DAI",
            "createdAt": "2019-08-13T19:12:27.386Z",
            "updatedAt": "2019-08-13T19:12:27.386Z",
            "deletedAt": null,
            "makerCurrencyUuid": "84298577-6a82-4057-8523-27b05d3f5b8c",
            "takerCurrencyUuid": "b656c441-68ab-4776-927c-d894f4d6483b",
            "makerCurrency": {
                "uuid": "84298577-6a82-4057-8523-27b05d3f5b8c",
                "symbol": "WETH",
                "contractAddress": "0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2",
                "decimals": 18,
                "soloMarket": 0,
                "createdAt": "2019-08-13T19:12:27.227Z",
                "updatedAt": "2019-08-13T19:12:27.227Z",
                "deletedAt": null
            },
            "takerCurrency": {
                "uuid": "b656c441-68ab-4776-927c-d894f4d6483b",
                "symbol": "DAI",
                "contractAddress": "0x89d24a6b4ccb1b6faa2625fe562bdd9a23260359",
                "decimals": 18,
                "soloMarket": 1,
                "createdAt": "2019-08-13T19:12:27.365Z",
                "updatedAt": "2019-08-13T19:12:27.365Z",
                "deletedAt": null
            }
        }
    ]
}
```

Returns:
`200` if successful

### POST /v1/dex/orders

Description:
Post a new order to the orderbook.

Headers:
```
Content-Type: application/json
```

Request Body:

|Field Name|JSON type|Description|
|----------|---------|-----------|
|order|Object|A valid signed order JSON object|
|fillOrKill|boolean|Whether the order should be canceled if it cannot be immediately filled|

Note: `fillOrKill` orders execute immediately and no part of the order will go on the open order
book. `fillOrKill` orders will either be completely filled, or not filled. Partial fills are not possible.

Example Request Body:
```json
{
	"fillOrKill": true,
	"order": {
		"makerMarket": "0",
		"takerMarket": "1",
		"makerAmount": "10000000000",
		"takerAmount": "20000000000",
		"makerAccountOwner": "0x3E5e9111Ae8eB78Fe1CC3bb8915d5D461F3Ef9A9",
		"makerAccountNumber": "111",
		"takerAccountOwner": "0x28a8746e75304c0780E011BEd21C72cD78cd535E",
		"takerAccountNumber": "222",
		"expiration": "4294967295",
		"salt": "100",
		"typedSignature": "0xd9c006cf9066e89c2e75de72604751f63985f173ca3c69b195f1f5f445289a1f2229c0475949858522c821190c5f1ec387f31712bd21f6ac31e4510d5711c2681f00"
	  },
};
```

Returns:
`201` if successful

### DELETE /v1/dex/orders/:hash

Description:
Cancels an open order by hash.

Please note you will need to provide a valid cancelation signature in the Authorization header in order to cancel an order.
The Authorization header signature should be hashed according to [EIP712](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md) and include the original orderHash but will not include any information about the order format, version, or chainId since these are already baked-into the hash of the order. You can see working examples of signing in the [LimitOrders](https://github.com/dydxprotocol/solo/blob/master/src/modules/LimitOrders.ts) module of Solo.js.

Headers:
```
Content-Type: application/json
Authorization: Bearer [A valid cancel signature]
```

Example Response Body:
```json
{
    "orders": [
        {
            "uuid": "d13aadc8-49fb-4420-a5a0-03c15b668705",
            "id": "0x2c45cdcd3bce2dd0f2b40502e6bea7975f6daa642d12d28620deb18736619fa2",
            "makerAccountOwner": "0x0913017c740260fea4b2c62828a4008ca8b0d6e4",
            "makerAccountNumber": "0",
            "status": "PENDING",
            "price": "1",
            "fillOrKill": false,
            "rawData": "{\"makerMarket\":\"0\",\"takerMarket\":\"1\",\"makerAccountNumber\":\"0\",\"takerAccountNumber\":\"222\",\"makerAccountOwner\":\"0x0913017c740260fea4b2c62828a4008ca8b0d6e4\",\"takerAccountOwner\":\"0x28a8746e75304c0780e011bed21c72cd78cd535e\",\"makerAmount\":\"10\",\"takerAmount\":\"10\",\"salt\":\"79776019296374116968729143546164248655125424402698335194396863096742023853053\",\"expiration\":\"0\",\"typedSignature\":\"0x9db8cc7ee2e06525949a0ae87301d890aee9973c464b276661d760ca8db4c73522ba48b94bf36d4aada7627656f79be9e40225a52f0adec079b07263b9e8ee0c1b01\"}",
            "makerAmount": "10",
            "unfillableAt": null,
            "unfillableReason": null,
            "takerAmount": "10",
            "expiresAt": null,
            "makerAmountRemaining": "10",
            "takerAmountRemaining": "10",
            "createdAt": "2019-07-29T23:56:25.522Z",
            "updatedAt": "2019-07-29T23:56:25.522Z",
            "deletedAt": null,
            "pairUuid": "b9b38876-c3a6-470e-81cf-d352d26685d0",
            "pair": {
                "uuid": "b9b38876-c3a6-470e-81cf-d352d26685d0",
                "name": "WETH-DAI",
                "createdAt": "2019-07-26T17:19:34.955Z",
                "updatedAt": "2019-07-26T17:19:34.955Z",
                "deletedAt": null,
                "makerCurrencyUuid": "84298577-6a82-4057-8523-27b05d3f5b8c",
                "takerCurrencyUuid": "b656c441-68ab-4776-927c-d894f4d6483b",
                "makerCurrency": {
                    "uuid": "84298577-6a82-4057-8523-27b05d3f5b8c",
                    "symbol": "WETH",
                    "contractAddress": "0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2",
                    "decimals": 18,
                    "soloMarket": 0,
                    "createdAt": "2019-07-26T17:19:34.627Z",
                    "updatedAt": "2019-07-26T17:19:34.627Z",
                    "deletedAt": null
                },
                "takerCurrency": {
                    "uuid": "b656c441-68ab-4776-927c-d894f4d6483b",
                    "symbol": "DAI",
                    "contractAddress": "0x89d24a6b4ccb1b6faa2625fe562bdd9a23260359",
                    "decimals": 18,
                    "soloMarket": 1,
                    "createdAt": "2019-07-26T17:19:34.919Z",
                    "updatedAt": "2019-07-26T17:19:34.919Z",
                    "deletedAt": null
                }
            },
            "fills": []
        }
    ]
}
```

### A note about Order and Fill status

Both orders and fills returned from the API will provide a status field.

For orders this field represents the current status of the order.
```javascript
export const STATUS = {
  PENDING: 'PENDING',
  OPEN: 'OPEN',
  FILLED: 'FILLED',
  PARTIALLY_FILLED: 'PARTIALLY_FILLED',
  CANCELED: 'CANCELED',
};
```

If the order was canceled, additional information will be provided by the `unfillableReason`
field.

For fills the status field represents the status of the transaction on-chain.

```javascript
export const STATUSES = {
  PENDING: 'PENDING',
  REVERTED: 'REVERTED',
  CONFIRMED: 'CONFIRMED',
};
```

### GET /v1/dex/orders

Description:
Get all open orders from the orderbook. This includes both unfilled and partially filled orders, but
does not include canceled, pruned, or unfillable orders.

Headers:
```
Content-Type: application/json
```

Query Params:

|Field Name|Description|
|----------|-----------|
|?makerAccountOwner|(Optional) The Ethereum address of the account(s) to request orders for.|
|?makerAccountNumber|(Optional) The Solo account number of the account to request orders for.|
|?limit|(Optional) The maximum number of orders to return. Defaults to 100.|
|?startingBefore|(Optional) ISO8601 string. Starts returning orders created before this date.|
|?pairs|(Optional) Array of pairs to filter by (e.g. ?pairs=WETH-DAI,DAI-WETH)|

Example Response Body:
```json
{
    "orders": [
        {
            "uuid": "d13aadc8-49fb-4420-a5a0-03c15b668705",
            "id": "0x2c45cdcd3bce2dd0f2b40502e6bea7975f6daa642d12d28620deb18736619fa2",
            "makerAccountOwner": "0x0913017c740260fea4b2c62828a4008ca8b0d6e4",
            "makerAccountNumber": "0",
            "status": "PENDING",
            "price": "1",
            "fillOrKill": false,
            "rawData": "{\"makerMarket\":\"0\",\"takerMarket\":\"1\",\"makerAccountNumber\":\"0\",\"takerAccountNumber\":\"222\",\"makerAccountOwner\":\"0x0913017c740260fea4b2c62828a4008ca8b0d6e4\",\"takerAccountOwner\":\"0x28a8746e75304c0780e011bed21c72cd78cd535e\",\"makerAmount\":\"10\",\"takerAmount\":\"10\",\"salt\":\"79776019296374116968729143546164248655125424402698335194396863096742023853053\",\"expiration\":\"0\",\"typedSignature\":\"0x9db8cc7ee2e06525949a0ae87301d890aee9973c464b276661d760ca8db4c73522ba48b94bf36d4aada7627656f79be9e40225a52f0adec079b07263b9e8ee0c1b01\"}",
            "makerAmount": "10",
            "expiresAt": null,
            "unfillableAt": null,
            "unfillableReason": null,
            "takerAmount": "10",
            "makerAmountRemaining": "10",
            "takerAmountRemaining": "10",
            "createdAt": "2019-07-29T23:56:25.522Z",
            "updatedAt": "2019-07-29T23:56:25.522Z",
            "deletedAt": null,
            "pairUuid": "b9b38876-c3a6-470e-81cf-d352d26685d0",
            "pair": {
                "uuid": "b9b38876-c3a6-470e-81cf-d352d26685d0",
                "name": "WETH-DAI",
                "createdAt": "2019-07-26T17:19:34.955Z",
                "updatedAt": "2019-07-26T17:19:34.955Z",
                "deletedAt": null,
                "makerCurrencyUuid": "84298577-6a82-4057-8523-27b05d3f5b8c",
                "takerCurrencyUuid": "b656c441-68ab-4776-927c-d894f4d6483b",
                "makerCurrency": {
                    "uuid": "84298577-6a82-4057-8523-27b05d3f5b8c",
                    "symbol": "WETH",
                    "contractAddress": "0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2",
                    "decimals": 18,
                    "soloMarket": 0,
                    "createdAt": "2019-07-26T17:19:34.627Z",
                    "updatedAt": "2019-07-26T17:19:34.627Z",
                    "deletedAt": null
                },
                "takerCurrency": {
                    "uuid": "b656c441-68ab-4776-927c-d894f4d6483b",
                    "symbol": "DAI",
                    "contractAddress": "0x89d24a6b4ccb1b6faa2625fe562bdd9a23260359",
                    "decimals": 18,
                    "soloMarket": 1,
                    "createdAt": "2019-07-26T17:19:34.919Z",
                    "updatedAt": "2019-07-26T17:19:34.919Z",
                    "deletedAt": null
                }
            },
            "fills": []
        }
    ]
}
```

### GET /v1/dex/fills

Description:
Get all historical fills.

Headers:
```
Content-Type: application/json
```

Query Params:

|Field Name|Description|
|----------|-----------|
|?makerAccountOwner|The Ethereum address of the account(s) to request fills for.|
|?makerAccountNumber|(Optional) The Solo account number of the account to request fills for.|
|?limit|(Optional) The maximum number of orders to return. Defaults to 100.|
|?startingBefore|(Optional) ISO8601 string. Starts returning orders created before this date.|
|?pairs|(Optional) Array of pairs to filter by (e.g. ?pairs=WETH-DAI,DAI-WETH)|

Example Response Body:
```json
{
    "fills": [
        {
            "uuid": "c389c0de-a193-49c3-843a-eebee25d1bfa",
            "messageId": "8f1ed6dc-8bd6-4155-ab33-5a252814f88b",
            "status": "PENDING",
            "orderId": "0x66a5b2d4bca3414ed902bd7cda0500df5947fadbfd48c280a206d44606c1c906",
            "transactionHash": "0x811cf67aca5fb8d085efcc47cd8213e767410866c7c840f2177391bf6e6b2fd0",
            "fillAmount": "10",
            "createdAt": "2019-07-27T00:48:15.963Z",
            "updatedAt": "2019-07-27T00:48:15.963Z",
            "deletedAt": null,
            "order": {
                "uuid": "b415de0d-a54c-4496-a8af-0a15d9fb95d5",
                "id": "0x66a5b2d4bca3414ed902bd7cda0500df5947fadbfd48c280a206d44606c1c906",
                "makerAccountOwner": "0x0913017c740260fea4b2c62828a4008ca8b0d6e4",
                "makerAccountNumber": "0",
                "status": "FILLED",
                "price": "1",
                "fillOrKill": false,
                "rawData": "{\"makerAccountOwner\":\"0x0913017c740260fea4b2c62828a4008ca8b0d6e4\",\"takerAccountOwner\":\"0x28a8746e75304c0780e011bed21c72cd78cd535e\",\"makerAccountNumber\":\"0\",\"takerAccountNumber\":\"222\",\"makerMarket\":\"0\",\"takerMarket\":\"1\",\"makerAmount\":\"10\",\"takerAmount\":\"10\",\"salt\":\"0\",\"expiration\":\"0\",\"typedSignature\":\"0xd9561c880b9572899eb97901f58423a610640357c1d36138f0bd31b16ca17edb715ec175b7cd7a308d70e88a6654ac706672419765720b6d8e357e60a9a5ce9b1c01\"}",
                "makerAmount": "10",
                "expiresAt": null,
                "unfillableAt": "2019-07-27T00:48:16.000Z",
                "unfillableReason": "ENTIRELY_FILLED",
                "takerAmount": "10",
                "makerAmountRemaining": "0",
                "takerAmountRemaining": "0",
                "createdAt": "2019-07-26T17:20:36.999Z",
                "updatedAt": "2019-07-27T00:48:16.001Z",
                "deletedAt": null,
                "pairUuid": "b9b38876-c3a6-470e-81cf-d352d26685d0",
                "pair": {
                    "uuid": "b9b38876-c3a6-470e-81cf-d352d26685d0",
                    "name": "WETH-DAI",
                    "createdAt": "2019-07-26T17:19:34.955Z",
                    "updatedAt": "2019-07-26T17:19:34.955Z",
                    "deletedAt": null,
                    "makerCurrencyUuid": "84298577-6a82-4057-8523-27b05d3f5b8c",
                    "takerCurrencyUuid": "b656c441-68ab-4776-927c-d894f4d6483b",
                    "makerCurrency": {
                        "uuid": "84298577-6a82-4057-8523-27b05d3f5b8c",
                        "symbol": "WETH",
                        "contractAddress": "0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2",
                        "decimals": 18,
                        "soloMarket": 0,
                        "createdAt": "2019-07-26T17:19:34.627Z",
                        "updatedAt": "2019-07-26T17:19:34.627Z",
                        "deletedAt": null
                    },
                    "takerCurrency": {
                        "uuid": "b656c441-68ab-4776-927c-d894f4d6483b",
                        "symbol": "DAI",
                        "contractAddress": "0x89d24a6b4ccb1b6faa2625fe562bdd9a23260359",
                        "decimals": 18,
                        "soloMarket": 1,
                        "createdAt": "2019-07-26T17:19:34.919Z",
                        "updatedAt": "2019-07-26T17:19:34.919Z",
                        "deletedAt": null
                    }
                }
            }
        },
    ]
}
```

## Accounts

### GET /v1/accounts/:address

Description:
Get account balances for a particular account owner

Headers:
```
Content-Type: application/json
```

Note: To get any account's collateralization, simply take `sumSupplyUsdValue / sumBorrowUsdValue`.
The minimum collateralization where liquidation occurs on the protocol using this formula is 1.15.

Query Params:

|Field Name|Description|
|----------|-----------|
|?number|(Optional) The Solo Acount number of the account to request balances for.|

Example Response Body:
```json
{
  "owner": "0x0913017c740260fea4b2c62828a4008ca8b0d6e4",
  "number": "0",
  "uuid": "72cd6a2a-17ff-4394-92d3-e951a96aa266",
  "hasAtLeastOneNegativePar": false,
  "hasAtLeastOnePositivePar": true,
  "sumBorrowUsdValue": "0",
  "sumSupplyUsdValue": "0",
  "balances": {
    "0": {
      "owner": "0x0913017c740260fea4b2c62828a4008ca8b0d6e4",
      "number": "0",
      "marketId": 0,
      "newPar": "9994719126810778",
      "accountUuid": "72cd6a2a-17ff-4394-92d3-e951a96aa266",
      "isParPositive": true,
      "isParNegative": false,
      "wei": "10000184397123234.892111593021043502",
      "expiresAt": null,
      "par": "9994719126810778",
      "adjustedSupplyUsdValue": "0",
      "adjustedBorrowUsdValue": "0"
    },
    "1": {
      "par": 0,
      "wei": 0,
      "expiresAt": null
    },
    "2": {
      "par": 0,
      "wei": 0,
      "expiresAt": null
    }
  }
}
```