---
description: >-
  Welcome to the Black Ocean API documentation. We provide seven different
  interfaces: Binary over TCP/IP and Websockets, JSON over TCP/IP and
  Websockets, Protobuf over TCP/IP and Websockets and REST.
---

# Introduction

## Testnet

Black Ocean features a Testnet environment, [_https://crms.bo.market_](https://crms.bo.market/), which can be used to generate a key and test the API. After registering, account access to the Testnet trading environment will be provided. For more information to get started, please visit [_https://bo.market/testnet-crm-guide_](https://bo.market/testnet-crm-guide/)_._

## Rate limits

Black Ocean does not place restrictions on how many requests can be sent but it employs a 1000 outstanding requests rule. Sending excess orders will result in a `REJECT` response.

