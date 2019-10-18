# thor-api-doc

# Authentication / Making requests

    API Base url: https://api.bitrefill.com/v1

**Our API uses HTTP Basic Auth for authentication.**

    Example: curl -u API_KEY:API_SECRET https://api.bitrefill.com/v1/test
    > Hello World!

# General flow

The general flow of a purchase is as follow: 

- 1) [Create an order](#order-creation)
- 2) Pay for it, send crypto to the generated address or use our [`/purchase` call](#purchase-call) to pay with your account balance
- 3) Check the order status either by [polling the order status](#query-order-status) or [using a webhook](#webhook)

# Inventory
Query inventory for packages in stock

GET /inventory/lightning-recharge

    Example: curl -u API_KEY:API_SECRET https://api.bitrefill.com/v1/inventory/lightning-recharge
    > {
      "operator": {
        "name": "Thor Recharge",
        "slug": "lightning-recharge",
        "isRanged": true,
        "currency": "SAT",
        "range": {
          "min": 500,
          "max": 4294967,
          "step": 1,
          "customerPriceRate": 9.739384389926184e-05,
          "customerSatoshiPriceRate": 1.0200634879994863,
          "customerEurPriceRate": 9.836778233825446e-05,
          "userPriceRate": 1.0200634879994863,
          "purchaseFee": 300.1106152450074
        },
    }


GET /inventory/lightning-channel

    Example: curl -u API_KEY:API_SECRET https://api.bitrefill.com/v1/inventory/lightning-channel
    > {
        "operator": {
          "name": "Thor Lightning Channel Capacity",
          "slug": "lightning-channel",
          "currency": "SAT",
          "range":{
            "min":500000,
            "max":16777215,
            "step":1,
            "customerPriceRate":0.0000010671007137192704,
            "customerSatoshiPriceRate":0.015185658220118399,
            "customerEurPriceRate":0.0000010777717208564632,
            "userPriceRate":0.015185658220118399,
            "purchaseFee":10023
          },
      }

GET /inventory/turbo-lightning-channel

Turbo channels include a balance and incoming capacity. Capacity will be double the value package selected.

    Example: curl -u API_KEY:API_SECRET https://api.bitrefill.com/v1/inventory/turbo-lightning-channel
    > {
      "operator": {
        "name": "Thor Turbo Lightning Channel",
        "slug": "turbo-lightning-channel",
        "currency": "SAT",
        "range": {
            "min":500000,
            "max":8388607,
            "step":1,
            "customerPriceRate":0.00007362994924662966,
            "customerSatoshiPriceRate":1.047072761487067,
            "customerEurPriceRate":0.00007436624873909596,
            "userPriceRate":1.047072761487067,
            "purchaseFee":10016.480236160778
        },
    }

# Order Creation

## Payment Method Lightning

POST /order

JSON Body Example

    {
        "operatorSlug": "lightning-channel",
        "valuePackage" : "2000000",
        "email" : "Customer email", // used for receipts
        "sendEmail" : true/false, // Optional. If false, receipt email won't be sent. Default: true
        "paymentMethod": "lightning" // Payment methods supported 'lightning' 'lightning-ltc' 'bitcoin' 'ethereum' litecoin' 'dash' 'dogecoin'
        "refund_address" : "1fdsfjakiwlewkld3845kd8", // Optional. If there is an error, we will send a refund to this address. Use bitcoin on chain addresses for lightning paymentMethod
        "webhook_url" : "http://your.webhook.url", // Optional
        "userRef": "internal-id-01" // Optional, a reference for your system ( max 128 chars )
    }


    Example: curl -u API_KEY:API_SECRET -H 'Content-Type: application/json' https://api.bitrefill.com/v1/order -d '{ "paymentMethod": "lightning", "operatorSlug": "lightning-channel", "valuePackage": "2000000", "email": "test@bitrefill.com"}'
    > {
        "id": "5ce472b9950b353c59bcd412",
        "email": "test@bitrefill.com",
        "expired": false,
        "value": "2000000",
        "product": "lightning-channel",
        "price": 78498,
        "partialPayment": false,
        "userRef": null,
        "status": "unpaid",
        "payment": {
          "address": "3MiyGH543zrv91TYst5tRLRcb3T1QTKFJq",
          "lightningInvoice": "lnbc784980n1pwwgu4epp5x60y2f09tvwl76gm8y86e9za69spvhhneh6xvscwnk8yg5fkencqdphgf5hgun9ve5kcmpqx43k2dphxf3rjwf4xp3rxdfnvv6njcnrvs6rzvscqzpgxqrp9sfppjm0qjpxwyj24dskmng2zr9gptumfewy8f8e8n2wfm99xvhc9ugyhaj7dxfqea4w05lgp3nc4k3c43hgsednvjf9d3yte3nhyd0qqw2nelaqdrqvmv4mmqrrwhn0dwt08j4www4pcqglxwv3",
          "satoshiPrice": 78498,
          "altcoinCode": "LNBC"
        }
      }

Lightning Recharge Order Example
    
JSON Body Example
    
    Lightning Invoice is set in the number property
    {
        "operatorSlug": "lightning-recharge",
        "number": "lnbc10u1pwjeee3pp53um2gnz6nsuflzkp77mjpmcurkmm5gvmv99yefhzxlt3dqkjqk7sdqqcqzpgqyg7tf54ddyusayg20m88ez02kn8uvu77rc95lcpetj3fqd68uajtl9xq0hsnupwqu68cxeqs5k7zj9gmkdt56k452x4u445yxgxm7sq5ljxkm",
        "valuePackage" : "1000", // satoshis
        "email" : "Customer email", // used for receipts
        "sendEmail" : true/false, // Optional. If false, receipt email won't be sent. Default: true
        "paymentMethod": "lightning" // Payment methods supported 'lightning' 'lightning-ltc' 'bitcoin' 'ethereum' litecoin' 'dash' 'dogecoin'
        "refund_address" : "1fdsfjakiwlewkld3845kd8", // Optional. If there is an error, we will send a refund to this address. Use bitcoin on chain addresses for lightning paymentMethod
        "webhook_url" : "http://your.webhook.url", // Optional
        "userRef": "internal-id-01" // Optional, a reference for your system ( max 128 chars )
    }

    Example: curl -u API_KEY:API_SECRET -H 'Content-Type: application/json' https://api.bitrefill.com/v1/order -d '{ "paymentMethod": "bitcoin", "operatorSlug": "lightning-recharge", "valuePackage": "1000", "email": "test@bitrefill.com","number": "lnbc10u1pwjeee3pp53um2gnz6nsuflzkp77mjpmcurkmm5gvmv99yefhzxlt3dqkjqk7sdqqcqzpgqyg7tf54ddyusayg20m88ez02kn8uvu77rc95lcpetj3fqd68uajtl9xq0hsnupwqu68cxeqs5k7zj9gmkdt56k452x4u445yxgxm7sq5ljxkm"}'
    >{
      "id": "5d2ce7ac33fce40004c7b781",
      "email": "test@bitrefill.com",
      "number": "lnbc10u1pwjeee3pp53um2gnz6nsuflzkp77mjpmcurkmm5gvmv99yefhzxlt3dqkjqk7sdqqcqzpgqyg7tf54ddyusayg20m88ez02kn8uvu77rc95lcpetj3fqd68uajtl9xq0hsnupwqu68cxeqs5k7zj9gmkdt56k452x4u445yxgxm7sq5ljxkm",
      "expired": false,
      "sent": false,
      "refunded": false,
      "value": "1000",
      "currency": "SAT",
      "operator": "lightning-recharge",
      "country": "international",
      "summary": "Thor Recharge 1000 SAT",
      "price": 1100,
      "commission": 0,
      "overcharge": 0,
      "priciness": 1.0404,
      "merchant_price": 1100,
      "satoshiPrice": 1100,
      "usdPrice": 0.12,
      "eurPrice": 0.1,
      "delivered": false,
      "partialPayment": false,
      "btcPrice": "0.000011",
      "itemDesc": "Thor Recharge 1000 SAT",
      "valuePackage": "1000",
      "operatorSlug": "lightning-recharge",
      "operatorType": "lightning",
      "invoiceTimeLeft": 899846,
      "invoiceTime": 1563223980090,
      "expirationTime": 1563224880090,
      "paymentMethod": "bitcoin",
      "zeroConfStatus": "pending",
      "recommended_fee_rate": 6,
      "payment": {
        "address": "3NonYQQ2HzZDc91rZK6n9VfCy6yv5N1TxN",
        "satoshiPrice": 1100,
        "BIP21": "bitcoin:3NonYQQ2HzZDc91rZK6n9VfCy6yv5N1TxN?amount=0.000011"
      }
    }

[Query order status result](#order-status-lightning-success)


## Payment Method balance

Create and Purchase Order when paying with bitcoin account balance

POST /purchase

JSON Body Example

    {
        "operatorSlug": "lightning-channel",
        "valuePackage" : "2000000",
        "email" : "Customer email", // used for receipts
        "sendEmail" : true/false, // Optional. If false, receipt email won't be sent. Default: true
        "webhook_url" : "http://your.webhook.url", // Optional
        "userRef": "internal-id-01" // Optional, a reference for your system ( max 128 chars )
    }


    Example: curl -u API_KEY:API_SECRET -H 'Content-Type: application/json' https://api.bitrefill.com/v1/purchase -d '{"operatorSlug": "lightning-channel", "valuePackage": "2000000", "email": "test@bitrefill.com"}'
    > {
        "id": "5ceca6d9cd48800004097403",
        "email": "test@bitrefill.com",
        "expired": false,
        "value": "2000000",
        "product": "lightning-channel",
        "price": 80900,
        "partialPayment": false,
        "userRef": null,
        "status": "payment_pending"
      }
      
[Query order status result](#order-status-balance-success)

# Query Order Status

GET /order/:orderid

After paying for an order, order status can be received through a webhook if webhook_url provided, or by manually querying order status.

## Order Status lightning Unpaid 

    Example: curl -u API_KEY:API_SECRET -H 'Content-Type: application/json' https://api.bitrefill.com/v1/order/5ce472b9950b353c59bcd412
    > {
        "id": "5ce472b9950b353c59bcd412",
        "email": "test@bitrefill.com",
        "expired": false,
        "value": "2000000",
        "product": "lightning-channel",
        "price": 78498,
        "partialPayment": false,
        "userRef": null,
        "status": "unpaid",
        "payment": {
          "address": "3MiyGH543zrv91TYst5tRLRcb3T1QTKFJq",
          "lightningInvoice": "lnbc784980n1pwwgu4epp5x60y2f09tvwl76gm8y86e9za69spvhhneh6xvscwnk8yg5fkencqdphgf5hgun9ve5kcmpqx43k2dphxf3rjwf4xp3rxdfnvv6njcnrvs6rzvscqzpgxqrp9sfppjm0qjpxwyj24dskmng2zr9gptumfewy8f8e8n2wfm99xvhc9ugyhaj7dxfqea4w05lgp3nc4k3c43hgsednvjf9d3yte3nhyd0qqw2nelaqdrqvmv4mmqrrwhn0dwt08j4www4pcqglxwv3",
          "satoshiPrice": 78498,
          "altcoinCode": "LNBC"
        }
      }

## Order Status lightning Success

    Example: curl -u API_KEY:API_SECRET -H 'Content-Type: application/json' https://api.bitrefill.com/v1/order/5ce472b9950b353c59bcd412
    > {
      "id": "5ce47aa5d4abef44c7a2f91e",
      "email": "test@bitrefill.com",
      "expired": false,
      "value": "2,000,000 sats capacity",
      "product": "lightning-channel",
      "price": 79184,
      "partialPayment": false,
      "userRef": null,
      "status": "success",
      "payment": {
        "address": "3GXtKpdmbeJUApgnkQ55VEb6mvePHEUzgj",
        "lightningInvoice": "lnbc784980n1pwwgu4epp5x60y2f09tvwl76gm8y86e9za69spvhhneh6xvscwnk8yg5fkencqdphgf5hgun9ve5kcmpqx43k2dphxf3rjwf4xp3rxdfnvv6njcnrvs6rzvscqzpgxqrp9sfppjm0qjpxwyj24dskmng2zr9gptumfewy8f8e8n2wfm99xvhc9ugyhaj7dxfqea4w05lgp3nc4k3c43hgsednvjf9d3yte3nhyd0qqw2nelaqdrqvmv4mmqrrwhn0dwt08j4www4pcqglxwv3",
        "satoshiPrice": 78498,
        "altcoinCode": "LNBC"
      },
      "thorInfo": {
        "link": "https://www.bitrefill.com/thor/7c2246f1e3df1656dd78727774ccc72c6a9de01a5c2b44ef0a924a36adb2978f",
        "k1": "7c2246f1e3df1656dd78727774ccc72c6a9de01a5c2b44ef0a924a36adb2978f",
        "lnurl": "lightning:lnurl1dp68gurn8ghj7ctsdyhxy6t5wfjkv6tvdshxxmmd9amrztm5dphhy0m385mkxv3jxsmxvvt9xdjxvvfkx5mxgeph8qmnydehxu6xxcmrxuexxdnp89jx2vp3vy6kxvnzxs6x2e3svyunydrpxvmxzerzxgunwwrxvzu77f",
        "other": "Open your new lightning channel with redemption instructions provided"
      }
    }
    
## Order Status Balance Success


    Example: curl -u API_KEY:API_SECRET -H 'Content-Type: application/json' https://api.bitrefill.com/v1/order/5ceca6d9cd48800004097403
    > {
      "id": "5ceca6d9cd48800004097403",
      "email": "test@bitrefill.com",
      "expired": false,
      "value": "2000000",
      "product": "lightning-channel",
      "price": 80900,
      "partialPayment": false,
      "userRef": null,
      "status": "success",
      "thorInfo": {
        "link": "https://www.bitrefill.com/thor/b14d6a2817733c9a1af2ec00c5d9b6bae60a7044ae840e68e2049adcd935e528",
        "k1": "b14d6a2817733c9a1af2ec00c5d9b6bae60a7044ae840e68e2049adcd935e528",
        "other": "Open your new lightning channel with redemption instructions provided"
      }
    }

## Webhook

If you have set up a webhook URL on order creation you will receive the same data as above directly to the provided URL once the channel is ready ( or failed to be created ). 

# Thor Data Response

When an order is successful, a generated QR code of the lightning lnurl can be shown, or a link to our site guiding the user to have a channel opened to them.

      "thorInfo": {
        "link": "https://www.bitrefill.com/thor/7c2246f1e3df1656dd78727774ccc72c6a9de01a5c2b44ef0a924a36adb2978f",
        "k1": "7c2246f1e3df1656dd78727774ccc72c6a9de01a5c2b44ef0a924a36adb2978f",
        "lnurl": "lightning:lnurl1dp68gurn8ghj7ctsdyhxy6t5wfjkv6tvdshxxmmd9amrztm5dphhy0m385mkxv3jxsmxvvt9xdjxvvfkx5mxgeph8qmnydehxu6xxcmrxuexxdnp89jx2vp3vy6kxvnzxs6x2e3svyunydrpxvmxzerzxgunwwrxvzu77f",
        "other": "Open your new lightning channel with redemption instructions provided"
      }
      
**link** 

Contains a link to our site containing instructions to redeem a channel

**lnurl**

Contains all data necessary for users to open a channel if lightning wallet supports lnurl.

Can be opened through a button via uri or can  be shown as a QR code.

LNURL can also be decoded to obtain channel opening capacity, push amount, node info.

**lnurl documentation**

More info on lnurl: https://github.com/btcontract/lnurl-rfc/blob/master/spec.md#1-incoming-payment-channel-request
