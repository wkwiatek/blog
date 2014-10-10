---
layout: post
title: "Building Backend-less Apps"
date: 2014-09-24 14:54
author:
  name: "Eugenio Pace"
  url:  http://www.twitter.com/eugenio_pace
  url: http://twitter.com/eugenio_pace
  avatar: https://secure.gravatar.com/avatar/702d07476c482418b948b911504137a5?s=60
tags:
- authentication
- Auth0 Lock
- Login
- SSO
---

**TL;DR**: you can securely access APIs from common providers like __AWS__, __Firebase__, __Azure Service Bus__, __Salesforce__, __Salesforce Communities__, __SAP Netweaver Gateway__, __Office 365__, from any app, using a simple interface, with no additional backend code.

---

Auth0 ships with built-in integrations with common APIs, used by many apps. Frequently, these APIs use slightly different or completely custom authentication mechanisms. Some use JWTs, others use their own proprietary tokens and schemes.

Currently Auth0 ships with built-in support for the following systems:

* **Amazon Web Services**
* **Azure Mobile Services**
* **Azure Service Bus**
* **Firebase**
* **Salesforce**
* **Salesforce Communities**
* **SAP Netweaver Gateway**

Auth0 makes it really easy to issue authentication tokens that are compatible with any of these systems, using a common interface. It takes care of all the idiosyncracies of each, so you can quickly build apps using these APIs.

![]()

More interestingly, this feature allows a very powerful use case: you can now build apps that don't require any backend code, because the token exchange is securely done by Auth0.

<!-- more -->

Example of this are: __Single Page Applications__ and __Mobile Systems__.

Also, becasue authentication of users is completely decoupled from the API, you can [use any Identity Provider](https://docs.auth0.com/identityproviders).

##How does it work?

Very easily. Let's see:

1. Configure the API integration. Auth0 provides complete tutorials for each.
2. Authenticate a user.
3. Call the `delegation` endpoint and get an API specific token.
4. Call your API.

Let's review each step in detail

###1. Configuration

Because each API is different, Auth0 includes detailed documentation for each of them. As an example, this is what's displayed with Azure Mobile Services:

![]()

Azure Mobile Services only requires entering their `Master Key` used to sign the JWT.

Integrating with Salesforce, requires a few extra paremeters:

![]()

###2. Authenticating the User

This is business as usual for an app using Auth0. A successful authenitcation will return a [JWT](https://docs.auth0.com/jwt). This token uniquely identifies the user in that app.

###3. Get the API specific authentication artifact

Here comes the magic. You call the `delegation` endpoint in Auth0's API including the `JWT` you got in step 2. It will look like this:

```
POST https://contoso.auth0.com/delegation
Content-Type: 'application/json'
{
  "client_id":   "8473hr3ir....iruir3nfdwee",
  "grant_type":  "urn:ietf:params:oauth:grant-type:jwt-bearer",
  "id_token":    "ey.....423434",
  "target":      "987423.....4234394743209",
  "api_type":    "wams",
  "scope":       "openid"
}
```

* `client_id` identifies the requesting app (e.g. your mobile app). This is the app for which the JWT was requested originally.
* `id_token` identifies the user you are requesting this on behalf-of.
* `target` parameter identifies this API endpoint in Auth0 (often the same as `client_id`.
* `api_type` identifies the type of API you are requesting this for (e.g. __Azure Mobile Services__ in the example above).

The response you get back is dependent of the `api_type`.

__Salesforce__ and __SAP__ use `access_tokens`:

```
{
  "scope": "full api",
  "instance_url": "https://na31.salesforce.com",
  "token_type": "Bearer",
  "access_token": "00Di0000000Xroy!AR0AQ....aYlYoJ8RxFE8QJeS2zi"
}
```

```
{
  "access_token":"AFBWmAGzHtKjj......rzwcxRGuOIdmH7",
  "token_type":"Bearer",
  "expires_in":"600",
  "scope":"SCOPE_IN_SAP"
}
```

__Azure Service Bus__ uses `Shared Access Signatures`:

```
{
  "azure_sb_sas": "SharedAccessSignature sig=k8bNfT81R8L...LztXvY%3D&se=14098336&skn=PolicyName&sr=http%3A%2F%2Fnamespace.servicebus.windows.net%2Fmy_queue"
}
```

In any case, you now have what is necessary to call the API.

###4. Calling the API

The final step is calling the API with the security artifact that Auth0 returned. Often, this means sending this on the `Authorization` header of your requests:

As an example, this request puts a message on a Windows Azure Service Bus queue:

```
POST https://my-app.servicebus.windows.net/my-queue/messages
Authorization: SharedAccessSignature sig=k8b...vY&se=14098336&skn=PolicyName&sr=http%3A%2F%2Fnamespace.servicebus.windows.net%2Fmy_queue
Content-Type: 'text/plain'

Hello from Auth0!
```

##Conclusion
