# Registrations

To link mobile app with Encedo HEM, a registration process needs to be performed. Whole process consists of 5 phases, where 1,2 and 5 are performed by HEM, whereas 3 and 4 by mobile app.

## Phase 1: Encedo subscription

First phase of generating pairing between Encedo and Mobile App it to subscribe to service by HEM. Data posted are first gathered from HEM directly and proxy here. 

```shell
curl -X POST "https://notify.encedo.com/subscribe"
     -H "Content-Type: application/json" 
     -d '{"eid": "jnLU9Bk2XW/J7ExvfID+xq89rIGyONQgO9bpXvCYKhs=", 
          "ses": "2ivAugFoMOyAShnurRsgIVvBWHNzuwbzB/iKJnnxmuM=", 
          "lbl": "John Doe (john@example.pl)"}'
```

> The above command returns JSON structured like this:

```json
{
	"url": "https://notify.encedo.com/register/70vt7sYLl6ZeMN71FCvJVL2K...O6jtLh7m70C",
	"id": "70vt7sYLl6ZeMN71FCvJVL2K...O6jtLh7m70C"
}
```

### HTTP Request
`POST https://notify.encedo.com/subscribe`


### POST Parameters
Parameter | Description
--------- | -----------
eid | Base64 Encedo public key
ses | Unique session key
lbl | Label to display on mobile app


### Response
Parameter | Description
--------- | -----------
url | Unique URL to render on QR code then to be scanned by mobile app
id | Case unique ID


### Status codes
Code | Meaning
---------- | -------
200 | OK
400 | Bad Request -- Missing arguments.


<aside class="success">
Received <code>&lt;url&gt;</code> should be rendered as QR code. Modile App can scan code and get encoded link directly in fast and convienient way for users.</aside>


<aside class="warning">
Parameters to be sent as POST data need to be retrieved from Encedo HEM.</aside>

Encedo HEM subsciption request looks like this:

```shell
curl "https://encedokey.com/api/2FA/subscribe"
     -H "Authorization: Bearer Advt7sYLl6ZeMN71FCvJVL2KO6jtLh7m70C="
```

> The above command returns JSON structured like this:

```json
{
	"eid": "jnLU9Bk2XW/J7ExvfID+xq89rIGyONQgO9bpXvCYKhs=",
	"lbl": "John Doe (john@example.pl)",
	"ses": "2ivAugFoMOyAShnurRsgIVvBWHNzuwbzB/iKJnnxmuM="
}
```














## Phase 2: Mobile app challenge

Mobile App will get direct link to this endpoint. Long and complicated <code>&lt;ID&gt;</code> can be retrived automatically. This endpoint is called by Mobiel Application only.

```shell
# This link will available via QR code scan
curl "https://notify.encedo.com/register/70vt7sYLl6ZeMN71FCvJVL2K...O6jtLh7m70C"
```

> The above command returns JSON structured like this:

```json
{
	"eid": "jnLU9Bk2XW/J7ExvfID+xq89rIGyONQgO9bpXvCYKhs=",
	"ses": "2ivAugFoMOyAShnurRsgIVvBWHNzuwbzB/iKJnnxmuM=",
	"lbl":  "John Doe (john@example.pl)",
}
```


### HTTP Request
`GET https://notify.encedo.com/register/<ID>`


### URL Parameters
Parameter | Description
--------- | -----------
ID | The ID retrive by QR code scan


### Response 
Parameter | Description
--------- | -----------
eid | Base64 Encedo public key
ses | Unique session key
lbl | Label to display on mobile app


### Status codes
Code | Meaning
---------- | -------
200 | OK
404 | Not Found -- The specified item could not be found.



<aside class="success">
HINT! Endpoint for this query is returned as <code>&lt;url&gt;</code> during phase 1. QR code scaner will return ready-to-call link.</aside>













## Phase 3: Mobile app response

Mobile App based on data received from Phase 2 and based on locally generated keys generates response for the Notify Service. This endpoint is called by Mobile Application only.

```shell
curl -X POST "https://notify.encedo.com/register/70vt7sYLl6ZeMN71FCvJVL2K...O6jtLh7m70C"
     -H "Content-Type: application/json"
     -d '{"aid": "n3StbWfb2YVVFmcDxzGek1RBXE9JS5eZVDVoRc77TFQ=",
          "res": "TDP+tvqJIaXYEGoJzETuLIIxAE8O9iv+KbfL0RAkAiE=",
          "fid": "secret_app_id"}'
```

> The above command returns JSON structured like this:

```json
{
	"url": "https://notify.encedo.com/event/",
	"pid": "98Me6Zf5iSrOXR4dS8KH9Kdhlg7ojXOTYBqFmaLeHZjs"
}
```


### HTTP Request
`POST https://notify.encedo.com/register/<ID>`


### URL Parameters
Parameter | Description
--------- | -----------
ID | The ID retrived by QR code scan


### POST Parameters
Parameter | Description
--------- | -----------
aid | Base64 encoded Mobile App public key
res | Digitaly signed <code>&lt;ses&gt;</code>
fid | Google FCM unique application push notification identyficator 


### Response
Parameter | Description
--------- | -----------
url | URL prefix for following queries performed by mobile app (more below)
pid | Pairing identificator, uniqly identyfing Encedo Hem with Mobile App (precisely: eid-aid) 


### Status codes
Code | Meaning
---------- | -------
200 | OK
400 | Bad Request -- Missing arguments.
404 | Not Found -- The specified item could not be found.


### How to generate <code>&lt;res&gt;</code>?

Mobile App generates curve25519 keypair: private part (<code>&lt;aid_prv&gt;</code> ) is stored inside the app, public part is called <code>&lt;aid&gt;</code>. Based on <code>&lt;eid&gt;</code> and <code>&lt;ses&gt;</code> response <code>&lt;res&gt;</code> is generated by following formula:

`key = ECDH( Base64Decode(aid_prv), Base64Decode(eid))`

`res = Base64Encode( HMAC(key, ses) )`






<aside class="success">
After this query, link between Encedo and Mobile App is setup but not confirmed. If posted <code>&lt;res&gt;</code> is correct, mobile application will receive final confirmation via FCM push-notification. Till then, link should be considered as 'pending'.</aside>




### Pair dataset store by app

For each pair, the app should store some data related to that pair. App needs to be able to lookup dataset by <code>&lt;pid&gt;</code> and <code>&lt;eid&gt;</code>.
 
Parameter | Description 
---------- | -------
pid | Pairing identificator, uniquely identyfing Encedo HEM
url | URL prefix for 'event'  queries
aid | Base64 encoded pair app public key (plus private part noted here as <code>&lt;aid_prv&gt;</code>)
fid | Google FCM unique application push notification identyficator (one common per app instance)
eid | Base64 Encedo public key
lbl | Label to display on mobile app











## Phase 4: Encedo status check

This endpoint called by subscriber is used, to check if Mobile Application finalizes registration process. 

```shell
# This link will available via QR code scan
curl "https://notify.encedo.com/subscribe/70vt7sYLl6ZeMN71FCvJVL2K...O6jtLh7m70C"
```

> The above command returns JSON structured like this:

```json
{
	"eid": "9ZMWP+1/b1UAaQ5OLvHy9hzrBdglPslur448amxtGBI=",
	"aid": "n3StbWfb2YVVFmcDxzGek1RBXE9JS5eZVDVoRc77TFQ=",
	"res": "TDP+tvqJIaXYEGoJzETuLIIxAE8O9iv+KbfL0RAkAiE=",
	"ses": "2ivAugFoMOyAShnurRsgIVvBWHNzuwbzB/iKJnnxmuM=",
	"ckey": "ILqmrYB9FmsUo7s56VIK0/htCPdzaLBN+N9di77zyHE="
}
```


### HTTP Request
`GET https://notify.encedo.com/subscribe/<ID>`


### URL Parameters
Parameter | Description
--------- | -----------
ID | Case unique ID


### Response
Parameter | Description
--------- | -----------
aid | Base64 encoded Mobile App public key
eid | Base64 Encedo public key
res | Digitaly signed  <code>&lt;ses&gt;</code>
ses | Unique session key
ckey | Unique session key, used to validate subscription session


### Status codes
Code | Meaning
---------- | -------
200 | OK
202 | Accepted
404 | Not Found -- The specified item could not be found.








## Phase 5. Encedo validation

Last step of generating pairing between Encedo and Mobile App is the final confirmation. This endpoint is called by subscriber.

```shell
curl -X POST "https://notify.encedo.com/subscribe/70vt7sYLl6ZeMN71FCvJVL2K...O6jtLh7m70C"
     -H "Content-Type: application/json"
     -d '{"hnd": "RPNrvSZIb+YE2CBZFlqpLpBgszJkAg6WysBzuVeQChU="}'
```

> The above command returns JSON structured like this:

```json
{
	"pid": "98Me6Zf5iSrOXR4dS8KH9Kdhlg7ojXOTYBqFmaLeHZjs"
}
```


### HTTP Request
`POST https://notify.encedo.com/subscribe/<ID>`


### POST Parameters
Parameter | Description
--------- | -----------
hnd | Handshake message


### Response
Parameter | Description
--------- | -----------
pid | Pairing identificator, uniquely identyfing Encedo Hem with Mobile App (precisely: eid-aid)


### Status codes
Code | Meaning
---------- | -------
200 | OK
400 | Bad Request -- Missing arguments.
401 | Unauthorized -- Validation failed.
404 | Not Found -- The specified item could not be found.


```shell
curl -X POST "https://encedokey.com/api/2FA/subscribe"
     -H "Authorization: Bearer Advt7sYLl6ZeMN71FCvJVL2KO6jtLh7m70C="
     -d '{"aid": "n3StbWfb2YVVFmcDxzGek1RBXE9JS5eZVDVoRc77TFQ=",
          "eid": "9ZMWP+1/b1UAaQ5OLvHy9hzrBdglPslur448amxtGBI=",
          "res": "TDP+tvqJIaXYEGoJzETuLIIxAE8O9iv+KbfL0RAkAiE=",
          "ckey": "9ZMWP+1/b1UAaQ5OLvHy9hzrBdglPslur448amxtGBI="
          "ses": "2ivAugFoMOyAShnurRsgIVvBWHNzuwbzB/iKJnnxmuM="}'
```

> The above command returns JSON structured like this:

```json
{
	"hnd": "RPNrvSZIb+YE2CBZFlqpLpBgszJkAg6WysBzuVeQChU="
}
```


<aside class="warning">Parameters to send as POST data need to be retrieved from Encedo HEM.</aside>
Encedo HEM subsciption validation request looks like this:


### FCM notification on success

<aside class="success">
Mobile app will receive via FCM confirmation generated by sending this structure:</aside>

>

```json
{
        "data": {
                "pairing": {
                        "pid": "<PID>",
                }
        },
        "to": "<FID>",
        "time_to_live": 60
}
```


## Other: Mobile app registration cancellation

Mobile App can cancel registartion e.g. when duplicated <code>&lt;eid&gt;</code> is detected, when already registered.

```shell
curl -X DELETE "https://notify.encedo.com/register/70vt7sYLl6ZeMN71FCvJVL2K...O6jtLh7m70C"
```

### HTTP Request
`DELETE https://notify.encedo.com/register/<ID>`


### URL Parameters
Parameter | Description
--------- | -----------
ID | The ID retrived by QR code scan


### Status codes
Code | Meaning
---------- | -------
200 | OK
404 | Not Found -- The specified item could not be found.




## Other: Subcription cancellation

Ongoing subscription process can be cancelled by HEM at any time. Confirmed subscription cannot be cancelled this way. Other endpoint needs to be used. 

```shell
curl -X DELETE "https://notify.encedo.com/subscribe/70vt7sYLl6ZeMN71FCvJVL2K...O6jtLh7m70C"
```

### HTTP Request
`DELETE https://notify.encedo.com/subscribe/<ID>`


### URL Parameters
Parameter | Description
--------- | -----------
ID | Case unique ID


### Status codes
Code | Meaning
---------- | -------
200 | OK
404 | Not Found -- The specified item could not be found.


