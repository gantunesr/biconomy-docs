---
sidebar_label: 'Paymaster API'
sidebar_position: 1

---

# Biconomy Paymaster Service


1. The Biconomy paymaster will enable a single paymaster URL generation on our dashboard, which can be used for gas sponsorship as well as enabling users to pay gas in ERC20 tokens.
2. Instead of keeping multiple paymaster URLs to differentiate the paymaster type, the API request body now includes a **MODE** parameter.
3. The **MODE** will be sent to indicate whether the dApp wants sponsored transactions for its users or wants to enable ERC20 payments by users.


### Paymaster Service APIs

The Paymaster Service will introduce two new endpoints for both paymasters mentioned above. Both endpoints must follow the [JSON RPC](https://www.jsonrpc.org/specification) specifications.

1. Get feeQuotes or Data Endpoint (applicable to Biconomy Hybrid Paymaster use case only)
2. Sponsor UserOp Endpoint


:::info
You can get your paymasterUrl from Biconomy Dashboard after registering the paymaster. The Paymaster will look like this
https://paymaster.biconomy.io/api/v1/{chainId}/[YOUR_API_KEY_HERE]

 The endpoint URL will same as paymasterURL for both `pm_getFeeQuotesOrData` and `pm_sponsorUserOperation`.

:::



### 1. Get Fee Quotes or Data Endpoint

**`pm_getFeeQuotesOrData`** This endpoint can be used to retrieve the **MODE**. The following cases are currently supported:

:::note

**"MODE"** is optional for `pm_getFeeQuotesOrData`

- If **"MODE"** in request is not defined, we initially check for sponsorship and return `paymaster&Data` (Sponsorship paymaster data) but if request cannot be sponsored, we return `FeeQuotes` (i.e. token paymaster data)

- If **"MODE"** is **SPONSORED** , we check for request sponsoreship. If request cannot be sponsored, we return the data as 0x. This would mean that user will have to pay for the gas fees. 

- If **"MODE"** is ERC20, we return Feequote for TokenPaymaster.
:::


#### 1. Mode is **SPONSORED**: 

> ***POST Request***

```javascript

{
	"jsonrpc": "2.0",
	"method": "pm_getFeeQuoteOrData",
	"id": 1,
	"params": [
		{
			...partialUserOp
		},
		{
			"mode": "SPONSORED",
      
      // Mandatory Fields
			"sponsorshipInfo": {
				"webhookData": {},
				"smartAccountInfo": {
					"name": "BICONOMY" | "INFINITSM",
					"version": "1.0.0"
				}
			}
		}
	]
}
```


In this case, since the mode is sponsored, `sponsorshipInfo` is required to decode the call data and check the limits set on the dashboard by the developer.

> ***Response***

 
```javascript
{
	"jsonrpc": "2.0",
	"id": 1,
	"result": {
		"MODE": "SPONSORED",
		"paymasterAndData": "0xdc91ffb7c4b800d70410a79a5b503ae4391f67e40000000000000000000000007306ac7a32eb690232de81a9ffb44bb346026fab00000000000000000000000000000000000000000000000000000000000000400000000000000000000000000000000000000000000000000000000000000041e1f74852c31150f18ef4e472b748148f8ae031849032218b26170414a18c9f99516eb13a4a9bd35d1334194348cccee3d270b6e7bb400b39f0c8d645266ead601c00000000000000000000000000000000000000000000000000000000000000"
	}
}
```

#### 2. Mode is **ERC20**:

> ***POST Request***

```javascript
{
	"jsonrpc": "2.0",
	"method": "pm_getFeeQuoteOrData",
	"id": 1,
	"params": [
		{
			..userOpParams
		},
		{
			"mode": "ERC20",
      // Mandatory Fields
			"tokenInfo": {
				"preferredToken": "0xbf22b04e250a5921ab4dc0d4ced6e391459e92d4",
				"tokenList": [
					"0xdA5289FCAAF71d52A80A254dA614A192B693e975",
					"0xda5289fcaaf71d52a80a254da614a192b693e977",
					"0xeabc4b91d9375796aa4f69cc764a4ab509080a58"
				]
			}
		}
	]
}
```

<details>
  <summary> Check for params here: </summary>

`paymasterAddress` *

Address of paymaster that is going to be used for given `UserOp`. In case of ERC20 mode, client should give approval to this address from user account.

`mode` *Optional*

This is the mode which is going to be used for given `UserOp`. This is only applicable for Hybrid Paymaster.

If value is **ERC20**, token paymaster flow will be used. In this case only `feeQuotes` are available in the response and `paymasterAndData` will not be present. Here, client can then show the list of tokens for user to choose the gas token or use the default token from response.

If value is **SPONSORED** , sponsorship paymaster flow will be used. Here paymasterAndData will be returned in the response and not the feeQuotes.

`paymasterAndData`  *Optional*

This field contains the `paymasterAndData` to be used in `UserOp` before signing on client side. This field is only returned if SponsorshipPaymaster can be used in the flow and mode value is **SPONSORED**.

`feeQuotes` *Optional*

This will be present in the response if mode is **ERC20**, it contains array of token objects that are applicable for gas payments for the given `UserOp`. This array is sorted by markup percentage.

**Fee Quotes Object Param explained**


| Parameter | Description |
| --- | --- |
| symbol | Token symbol |
| decimal | Token decimal |
| address | Token address |
| maxGasFee | Max amount of gas fee in token units that can be charged from the user account. This includes markup fee and additional cost of token approval. |
| logoUrl | Token logo URL |
| exchangePrice | Exchange price of token. No of tokens in 1 unit of native currency. |
| markupPercentage | Markup percentage that’ll be charged for given token on top of gas fee. |


</details>


> ***Response***

```javascript
{
	"jsonrpc": "2.0",
	"id": 1,
	"result": {
		"MODE": "ERC20",
		"paymasterAddress": "0x716bc27e1b904331c58891cc3ab13889127189a7",
		"feeQuotes": [
			{
				"symbol": "USDT",
				"decimal": 18,
				"tokenAddress": "0xbf22b04e250a5921ab4dc0d4ced6e391459e92d4",
				"maxGasFee": 0.25267043311569887,
				"maxGasFeeUSD": 0.0010842277144671015,
				"exchangeRate": 232808808808808800000,
				"premiumPercentage": "13",
				"validUntil": 1686847490,
				"logoUrl": "https://raw.githubusercontent.com/spothq/cryptocurrency-icons/master/128/color/usdt.png"
			}
		],
		"unsupportedTokens": [
			"0xbf22b04e250a5921ab4dc0d4ced6e391459e92d3",
			"0xda5289fcaaf71d52a80a254da614a192b693e977",
			"0xeabc4b91d9375796aa4f69cc764a4ab509080a58"
		]
	}
}
```

:::info
If the paymaster mode is **ERC20** then it is recommended to include the `tokenInfo` field in order to avoid latency.

- If `preferredToken` is included, fee quotes will return a response only for this token.

- If `preferredToken` is blank, `tokenList` will be considered and feeQoute for all mentioned tokens in the array will be returned.

- If Both `preferredToken` & `tokenList` are empty, `feeQuote` for all the supported token by Biconomy will be returned in the response.
:::




:::note
If both `preferredToken` & `tokenList` are present in the request body, but some tokens are not supported by Biconomy, those will be returned also in the response body under `unsupportedTokens`.
:::


#### 3. Mode is **EMPTY** :

In this case, the API should check for both SponsorshipPaymaster and Token Paymaster conditions. If the `mode` parameter is not available in the request, it will start checking the SponsorshipPaymaster conditions. If these conditions are met, it will return the response with `paymasterAndData` but without the `feeQuotes`. The `mode` value here must be `SPONSORED`.

If the Sponsorship Paymaster conditions fail, then it will check for Token Paymaster conditions as described in the TokenPaymaster section, and return the response accordingly. In this case feeQuotes is returned but not the paymasterAndData.

However, if the `mode` value is available in the request and is equal to `ERC20`, then checks for Sponsorship Paymaster should be skipped, and only the Token Paymaster flow should be followed.

Based on the `mode` value in the response, the client will decide whether to show gas payment options to the user or not.

> ***POST Request***

```javascript
{
	"jsonrpc": "2.0",
	"method": "pm_getFeeQuotesOrData",
	"id": "1",
	"params" : {
		{
		// Partial UserOp object
		},                        
		{
			"tokenInfo": {
				"preferredToken": "0xbf22b04e250a5921ab4dc0d4ced6e391459e92d3",
				"tokenList": [
					"0xbf22b04e250a5921ab4dc0d4ced6e391459e92d4",
					"0xda5289fcaaf71d52a80a254da614a192b693e977",
					"0xeabc4b91d9375796aa4f69cc764a4ab509080a58"
				]
			},
			"sponsorshipInfo": {
				"webhookData": {},
				"smartAccountInfo": {
					"name": "BICONOMY",
					"version": "1.0.0"
				}
			}
		}
	}
}
```

> ***Response***

- **Success Response** :

Either one of the two above mentioned cases, whichever is successful.

- **Error Response**

```javascript

{
	"jsonrpc": "2.0",
	"id": "1",
	"error": {
		"code": ERROR_CODE,
		"message": "error message"
	}
}
```

---

### 2. Sponsor UserOP End Point

**`pm_sponsorUserOperation`** : 

This API is responsible for calculating the `paymasterAndData` field, if applicable, for the given request.

The API receives a partial `UserOp` and an optional token address. From the mode parameter in request, it checks for the type of paymaster this request is for, and processes the request accordingly, as outlined below.

**Sponsorship Paymaster**

To determine whether a request can be sponsored by the Sponsorship Paymaster, only consider the partial UserOp object and the sponsorshipInfo. If the request cannot be sponsored, return "0x" as the `paymasterAndData`.

**Token Paymaster**

Consider both the partial `UserOp` and `tokenAddress` parameters in this case. If the `tokenAddress` is absent, an error will be thrown. If both are present, check for the *TokenPaymaster* flow and send the `paymasterAndData` response accordingly. If the `tokenAddress` is not supported, an error will also be thrown. Please see the error response at the end of this document.


:::note
**"MODE"** is mandatory for `pm_sponsorUserOperation` API, 

- If **"MODE"** is **SPONSORED**, we check for request sponsorship. If request cannot be sponsored, we return 0x. This would mean users will pay for their gas fees.
- If **"MODE"** is ERC20, we return `paymasterAndData` for TokenPaymaster. This would mean that users will pay in there preferred ERC20 Tokens
:::

#### 1. Mode is **ERC20** :

> ***POST Request***

```javascript
{
	"id": 1,
	"jsonrpc": "2.0",
	"method": "pm_sponsorUserOperation",
	"params": [
		{
			sender, // address
      nonce, // uint256
      initCode, // string
      callData, // string
      callGasLimit, // string
      verificationGasLimit, // string
      preVerificationGas, // string
      maxFeePerGas, // string
      maxPriorityFeePerGas, // string
		},
		{
			"mode": "ERC20",
			"tokenInfo": {
				"feeTokenAddress": "0xbf22b04e250a5921ab4dc0d4ced6e391459e92d4"
			}
		}
	],
}

```

<details>
  <summary> Check for params here: </summary>

`params` is an array where 

| Index | Description |
| --- | --- |
| 0 | Partial `UserOp` without signature and `paymasterAndData` |
| 1 | Context data containing extra information based on type of paymaster used.

`mode`
Mandatory Field. Values can be SPONSORED or ERC20

`tokenInfo`
In case of MODE is ERC20, `tokenInfo` needs to be present in the request body.

`sponsorshipInfo`
In case of MODE is SPONSORED, `sponsorshipInfo` needs to be present in the request body.
*Webhook is optional.*
*SmartAcountInfo is Mandatory.* |


  </details>

> ***Response***

**Success Response**

```javascript

{
	"jsonrpc": "2.0",
	"id": 14,
	"result": {
		"paymasterAndData": "0xsdfsdfa000000asas..."
	}
}
```

In response, the result field will contain `paymasterAndData` to be used in UserOp.

:::note

In the above REQUEST, the user has the option to include **`calculateGasLimits`** parameter that allows us us to *calculate the gas specific fields from userOp* again. This can be set to either **`true`** or **`false`** . When **`calculateGasLimits`**  is not explicitly included in the request, it defaults to **`false`** which is the case in above REQUEST and the value returned is ` "paymasterAndData: "0xsdfsdfa000000asas..."` .

If the value was **`true`**, this is how the **REQUEST** and **RESPONSE** would look like:

> ***POST Request:***

```javascript
{
	"jsonrpc": "2.0",
	"id": 14,
	"method": "pm_sponsorUserOperation",
	"params": [
		{ ...partialUserOp },
		{
			"calculateGasLimits": true,
			"mode": "ERC20",
			"tokenInfo": {
				"feeTokenAddress": "0xfd086bc7cd5c481dcc9c85ebe478a1c0b69fcbb9"
			}
		}
	]
}
```

> ***Response:***

```javascript
{
	"jsonrpc": "2.0",
	"id": 14,
	"result": {
		"preVerificationGas": "2320098",
		"verificationGasLimit": 2647740,
		"callGasLimit": 600000,
		"paymasterAndData": "0xsdfsdfsdfsdf....."
	}
}
```
:::



#### 2. Mode is **SPONSORED** :

> ***POST Request***

```javascript

{
	"id": 1,
	"jsonrpc": "2.0",
	"method": "pm_sponsorUserOperation",
	"params": [
		{
			sender, // address
      nonce, // uint256
      initCode, // string
      callData, // string
      callGasLimit, // string
      verificationGasLimit, // string
      preVerificationGas, // string
      maxFeePerGas, // string
      maxPriorityFeePerGas, // string
		},
		{
			"mode": "SPONSORED",
			"sponsorshipInfo": {
				"webhookData": {},
				"smartAccountInfo": {
					"name": "BICONOMY",
					"version": "1.0.0"
				}
			}
		}
	]
}
```

> ***Response***

**Success Response**

```javascript

{
	"jsonrpc": "2.0",
	"id": 14,
	"result": {
		"paymasterAndData": "0xsdfsdfa000000asas..."
	}
}
```


In response, the result field will contain `paymasterAndData` to be used in UserOp.

