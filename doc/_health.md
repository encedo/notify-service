# Health check

Anti-fraud feature is fully anonymous! Every power-up or every 24h, HEM generates unique session ID and hash of this ID via HMAC session ID with Device ID. This simple algorithm allows remote wipeout of the HEM (e.g. when reported as stolen) by reporting Device ID. There is no way to track not reported HEM (Device ID cannot be discovered).


## Health check a.k.a. anti-fraud system + firmware version check

Encedo HEM call this endpoint to verify fraud status. Firmware version can be also checked (option) by providing firmware version hash id. This endpoint should be called frequently by HEM local management application.


```shell
curl -X POST "https://notify.encedo.com/health"
     -H "Content-Type: application/json"
     -d '{"sessid": "jnLU9Bk2XW/J7ExvfID+xq89rIGyONQgO9bpXvCYKhs=",
          "sessdevid": "2ivAugFoMOyAShnurRsgIVvBWHNzuwbzB/iKJnnxmuM=",
          "fw_pubid": "1EEfqdXLb5G3vRjkq0Hn0Dvzfh1zjBK5I+8PCd6Q5s8=",
          "bl_pubid": "AdGNYPXnNUpP9HQWMDKQQgHRjmDt5zFKT/B0Z0gykEI="}
```

> The above command returns JSON structured like this (no new FW version, HEM not reported as stolen):

```json
{
	"fw_version": "current",
	"bl_version": "current"
}
```

> The above command returns this when newer version of applicable FW is available:

```json
{
	"fw_version": "BNqLf3bo1R9dnJuDyJS+W8Bwj3rqKTSaVPDgBp85ZgA=",
	"bl_version": "current"
}
```

> In this case, HEM is marked as stolen and remote action 'wipeout' should be performed:

```json
{
	"fw_version": "current",
	"bl_version": "current",
	"action": "wipeout",
	"sig": "i+7QJDUbmmQ2Ek2Xmgq2lXrAQvCL1BGuA1ChfgMGnU4="
}
```


### HTTP Request

`POST https://notify.encedo.com/health`

### POST Parameters

Parameter | Description
--------- | -----------
sessid | Random Base64 encoded session id
sessdevid | sessid HAMC by Device ID
fw_pubid | (optional) Hash of the firmware version
bl_pubid | (optional) Hash of the bootloader version



### Response
Parameter | Description
--------- | -----------
fw_version | Status of the firmware version ('current', 'uknown' or hash of new version)
bl_version | Status of the bootloader version (like above)
sig | Signed Device ID and sessid by Device bootloader private key
action | Action to perform on reported HEM ('wipeout', 'lock', 'track')

### Status codes
Code | Meaning
---------- | -------
200 | OK
400 | Bad Request -- Missing arguments.




<aside class="warning">Parameters to send as POST data needs to be retrieve from Encedo HEM.</aside>

This Encedo HEM endpoint provides necessery data:

```shell
curl "https://encedokey.com/api/health"
```

> The above command returns JSON structured like this:

```json
{
	"sessid": "9QPHq+j9vzSAGu3Prjpcrqudnxzup0hgKS01m11bw50=",
	"sessdevid": "0wnnmo2Mdmo/VX5er483DtnoKEWi7gj/0l65hd9s+ck=",
	"bl_pubid": "AdGNYPXnNUpP9HQWMDKQQgHRjmDt5zFKT/B0Z0gykEI=",
	"bl_sign": "/////y3p8EMBaCHwAQEBYAAhAWBBYIFgwWABYSEhQWE+ST5KEDlP8D0IkEID0cH4CIC96PCDOUpP9HRlGDKQQg==",
	"fw_pubid": "1EEfqdXLb5G3vRjkq0Hn0Dvzfh1zjBK5I+8PCd6Q5s8=",
	"fw_sign": "j2yoUKYGdmhJKTj74Zb+2SwK3Q2TBzD5a4JXOK7QOLYc1HSDfgyECHqAS+JTB8LHwUEZ3AXSM4A2rKtmMFNzAQ==",
	"integtity_check": "OK"
}
```


<aside class="warning">Health check status should be routed back to encedo HEM.</aside>

> This Encedo HEM endpoint accept health status:

```shell
curl -X POST "https://encedokey.com/api/health"
     -H "Content-Type: application/json"
     -d '{"fw_version": "current", "bl_version": "current",
        "action": "wipeout", "sig": "i+7QJDUbmmQ2Ek2Xmgq2lXrAQvCL1BGuA1ChfgMGnU4="}'
```

