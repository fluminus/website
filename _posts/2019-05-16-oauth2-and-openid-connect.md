---
layout: post
title: OAuth2 and OpenID Connect
date: 2019-05-16 17:45 +0800
categories: fluminus
---

This is from the development of (luminus_api)[https://github.com/fluminus/luminus_api], when I was dealing with the OpenID Connect authentication flow.

<!--more-->

# OAuth 2 and OpenID Connect

**Delegated authorization**: How can I let a website access my data without giving my password? 

![](https://i.imgur.com/vAp68JZ.png)

## OAuth 2.0 terminology

- Resource owner: the user who provides credentials (click 'yes I allow this access')
- Client: the application in need of user's data
- Authorization server: 'accounts.google.com'
- Resource server: where the actual user data is stored (can be the same system as Authorization server, but mostly seperate)
- Authorization grant: proof that the `resource owner` approves `client`'s access to the data
- Redirect URI: if the user clicks yes, where the user will be lead to
- Access token: the key for the `client` to access the data it needs
- Scope: access scope that `client` requests
- Consent: whether the `resource owner` gives `consent` to the `scope`

![](https://i.imgur.com/rOPAX4m.png)

*Why get a code instead of an access token right away?*

- Back channel (highly secured channel, dashed lines): connection from a server to the destination
    - When exchanging auth code for access code from the auth server, we append some sort of secret key that's available only to your own server to prevent auth code stealing
- Front channel (not as secure, solid lines): connection from the browser
    - Used for interacting with the user (i.e. requesting for credentials, consent for scopes)

## Starting the flow

![](https://i.imgur.com/VTeZd25.png)

When registering your org with the auth provider, you're given `client_id` and `client_secret`

![](https://i.imgur.com/Rxplwqs.png)

## OAuth 2.0 flow types

- Authorization code (front channel + back channel)
- Implicit (front channel only)
    - ![](https://i.imgur.com/QcpPU2B.png)
- Resource owner password credentials (back channel only)
- Client credentials (back channel only)

## Problems with OAuth 2.0 for **authentication**

- No standard way to get the user's information
- Every implementation is a little different
- No comment set of scopes

(Then people invent a layer above OAuth named OpenID Connect for authentication use cases for OAuth)

- OpenID Connect for **authentication**
- OAuth 2.0 for **authorization**

## What OpenID Connect adds

- ID token
    - a JSON Web Token (JWT)
    - Header
    - Payload(claims)
    - Signature: verify the `id_token` is authentic and not modified
- userinfo endpoint for getting more user information
    - `id_token` may not contain all the information needed
    - go to userinfo endpoint with access token for more info
- standard set of `scope`s
- standardized implementation

![](https://i.imgur.com/pidGkl2.png)

(The only difference is the additional **openid** scope)

## OAuth and OpenID Connect

### Use OAuth 2.0 for **Authorization**

- Grant access to your API
- Getting access to user data in other systems

### Use OpenID Connect for **Authentication**

- Logging the user in
- Making you accounts available in other systems

A library for best practices in OAuth2 and OpenID Connect [AppAuth](https://appauth.io/)