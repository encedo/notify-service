---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell: cURL

toc_footers:
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - register
  - notify
  - health
  - errors

search: true
---

# Introduction

Welcome to the Encedo Notification Services API! Notification Service is a broker type service, acting as a proxy between Encedo HEM and Managment Mobile Application. Main purpose of the service is to allow interchange of data between HEM and Mobile Apps that allow 2FA, remote managment and governance endpoint.


Encedo Notification Services is based on RESTful API. Server does not hold any critical or confidential data e.g. private keys or personal data. The only piece of data stored on servers' disks are public keys of HEM and Mobile App, and the association between them.

Source code of this API is available on [GitHub](https://github.com/encedo/notify-services). Feel free to clone, review, criticise or propose new features or enhancements. If you have found a bug, please follow responible discosure practice and report details to us first via bugbounty@encedo.com. Expect reward!




NOTES

ECDH(prv,pub) - curve25519 ECDH function

HMAC(key,data) - SHA256 based HMAC function

Base64Decode(str) - base64 string to binary decoder

Base64Encode(bin) - base64 binary to string encoder


# Authentication

Access to Notification Services is authorized based on digital signature over challenge-response scheme. More details about this mechanism is described below together with related endpoints descriptions.


