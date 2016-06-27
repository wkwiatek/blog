---
layout: post
title: "How to convince your boss to use Auth0"
description: Learn how to convince your technical boss to use Auth0.
date: 2016-06-27 5:00
author:
  name: Prosper Otemuyiwa
  url: https://twitter.com/unicodeveloper?lang=en
  avatar: https://en.gravatar.com/avatar/1097492785caf9ffeebffeb624202d8f?s=200
  mail: prosper.otemuyiwa@auth0.com
design:
  bg_color: "#4A4A4A"
  image: https://cdn.auth0.com/blog/laravel-auth/logo.png
tags:
- serverless
- backendless
- webtask
- lambda
- functions
- extensibility
- nodejs
- platform
- saas
- paas
- webhook
related:
- 2015-10-07-extensibility-through-code-using-webtasks
- 2015-12-11-get-your-twitter-share-count-back-with-a-webtask
- 2015-07-28-if-this-then-node-dot-js-extending-ifttt-with-webtask-dot-io

---

Auth0 is an enterprise-grade platform for modern identity. We provide tools that eliminate the friction of authentication for your applications and APIs. Our products increase developer productivity by at least 50 percent while ensuring security is top-notch. Use these points below to explain the advantages of using Auth0 in your development workflow to your company.

## The Benefits of Using Auth0

There are many benefits to leveraging Auth0 in your software development process.

### 1 - Single Sign On

You can enable Single Sign On for your corporate applications like Salesforce, Dropbox, Slack, and many more! Auth0 provides out-of-the-box support for more than 15 cloud applications. These applications are: Microsoft Azure Active Directory, Box, CloudBees, Concur, Dropbox, Microsoft Dynamics CRM, Adobe Echosign, Egnyte, New Relic, Office 365, Salesforce, Sharepoint, Slack, Springcm, Zendesk, and Zoom.

Implementing enterprise SSO is one of the easiest ways to take your SaaS upmarket and grow your revenue. Enabling enterprise clients to allow their employees to login to your application with their company details is likely a necessity for a potential enterprise customer to choose your SaaS. Implementing SAML authentication in Auth0 is as easy as adding a few lines of code and adding your SAML identity provider information into the dashboard like so:

<iframe src="//fast.wistia.net/embed/iframe/2xrll0d056" allowtransparency="true" frameborder="0" scrolling="no" class="wistia_embed" name="wistia_embed" allowfullscreen mozallowfullscreen webkitallowfullscreen oallowfullscreen msallowfullscreen width="640" height="400"></iframe>
<script src="//fast.wistia.net/assets/external/E-v1.js" async></script>

What if your application is not in the list of our pre-integrated apps? No problem! We also support the main industry standards such as SAML, WS-Fed, and OAuth 2.0 so you can hook any third-party application that you need.

Auth0 also provides **Single Log Out**. The ability for you to log out of one of your applications, hence logging you out in all of them.

### 2 -  Social Login

There are so many platforms currently that requires some form of registration to have access to. According to a report by [Web Hosting Buzz Survey](http://www.webhostingbuzz.com/blog/2013/03/21/whos-sharing-what/), more than 80 percent of users are bothered about creating new accounts on websites. So, providing social login to your apps will increase the number of registrations on your platform.

Auth0 supports 30+ social providers: Facebook, Twitter, Google, Yahoo, Windows Live, LinkedIn, GitHub, PayPal, Amazon, vKontakte, Yandex, 37signals, Box, Salesforce, Salesforce (sandbox), Salesforce Community, Fitbit, Baidu, RenRen, Weibo, AOL, Shopify, WordPress, Dwolla, miiCard, Yammer, SoundCloud, Instagram, The City, The City (sandbox), Planning Center, Evernote, Evernote (sandbox), and Exact. Additionally, you can use Auth0’s Connections API to add any OAuth2 Authorization Server as an identity provider. ​Every provider has its own profile properties, required headers, and response format, and some use OAuth1 (Twitter) while others use OAuth2.

_Social Login with Auth0 in a few clicks_

<img src="https://cdn.auth0.com/content/social-login/enabling-social-providers-2.gif" alt="Enabling social providers" />

### 3 - Fingerprint Authentication

Users are always faced with the challenge of remembering a number of different passwords, they want the simplest means of secure authentication as possible. One way to combat that challenge is to leverage on the fingerprint sensors that are now prevalent on mobile devices. **Auth0** provides the platform to authenticate users with **TouchID**. When a user signs up to use fingerprint authentication, Auth0 generates a key pair on the device, a public key and a private key. The user will be created in Auth0, the public key will be registered for the user, and the private key is stored in the keystore of the device.

<img src="https://auth0.com/learn/wp-content/uploads/2016/04/passwordless-hero.png" alt="Fingerprint Authentication with TouchID" />

_How Auth0 Fingerprint works_

<img src="https://auth0.com/learn/wp-content/uploads/2016/04/Fingerprint-Diagram.png" alt="How Auth0 Fingerprint works" />

When a user tries to authenticate with a valid fingerprint, the following processes occur:

* Touch ID retrieves the private key from the keystore
* A JSON Web Token is Created
* This JWT is then signed with the private key
* The app sends this signed JWT to Auth0
* Auth0 returns an **id_token**, the user profile and, optionally, a **refresh_token**


### 4 - Multi-factor authentication

Several applications suffer different level of authentication attacks. One way to eradicate such attacks is to require users to present more than one piece of identifying information. **Auth0** multi-factor authentication is a method of verifying users which requires them to provide more than just username and password. It provides a second layer of security to user log-ins and transactions.

Why use Auth0 Multi-factor Authentication?

* **Easy to use** - Auth0 Multi-factor Authentication is simple to setup and use. Find out more [here](https://auth0.com/docs/multifactor-authentication)
* **Scalable** - Auth0 Multi-factor Authentication harnesses the power of the cloud, hence supporting high volume critical scenarios
* **Always protected** - Auth0 Multi-Factor Authentication provides strong authentication using the highest industry standards
* **Reliable** - 99.98% uptime guarantee of Auth0 Multi-factor Authentication

**Auth0** allows you to customize and extend the authentication flow through JavaScript functions called [rules](https://auth0.com/docs/rules). **Auth0** provides rule templates to speed the creation of new rules and a large number of useful rules are have been contributed by the active community on GitHUB (https://github.com/auth0/rules).

<img src="https://cdn.auth0.com/content/email-wall/use-cases/mfa-for-customers/auth-pipeline-with-rules-customers.png" alt="Auth pipeline with rules for customers" />

Rules are run after the customer is authenticated and before control is returned to the application.

**Auth0** provides [2-Factor Authentication](https://auth0.com/learn/two-factor-authentication/) with **Google Authenticator** and **Duo** out of the box.

### 5 - Passwordless Authentication

Passwordless Authentication allows users to login without the need to remember a password. This improves user experience greatly since users will only need an email address or phone number to register for your application. It cuts down registration time by half, thus allowing more users to easily register on your application/platform. With **Auth0**, you can set up passwordless authentication on Single Page Apps, Regular Web Apps, iOs and Android Apps. **Auth0** provides various options for achieving passwordless connections as shown below:

<img src="https://cdn.auth0.com/docs/media/articles/connections/passwordless/passwordless-connections.png" alt="Auth0 passwordless authentication" />

**Auth0** provides the ability to build your own custom passwordless implementation. Check out the [complete API reference](https://auth0.com/docs/auth-api#passwordless).

You can read more about **Auth0** secure passwordless connections [here](https://auth0.com/docs/connections/passwordless).

### 6 - Serverless

Serverless is a new cloud computing trend that changes the way applications are being built and maintained. Serverless ensures you focus on code, not servers. Traditionally, a software engineer has to write code, spin up servers, ensure the servers are communicating with each other effectively and also be concerned with code deployment to the servers.

**Auth0** provides the ability for software engineers to solely focus on code and business logic with [webtask](https://webtask.io/) without bothering about the server infrastructure.

<img src="https://cdn.auth0.com/blog/serverless/webtasks.png" alt="Auth0 Webtask" />

At **Auth0**, we focus on running, provisioning, maintaining and scaling the general infrastructure for you while you think up ideas and write code. We have been using webtasks to support [platform extensibility through custom code](https://auth0.com/blog/2015/10/07/extensibility-through-code-using-webtasks/). Webtasks allow us to execute untrusted code with very low latency in a secure way, which is the foundation of a key customization feature we offer our customers. We like to think about webtasks as a much more developer friendly alternative compared to other vendors, and one that offers great flexibility thanks to high fidelity to HTTP. You can use webtasks in a variety of applications, from sandboxing untrusted code, to creating API gateways, lightweight HTML applications, or API backends for mobile apps.

With **Auth0 webtask**, A developer only pays for the actual time functions are run in the codebase. Check out a sample app built using webtask [here](https://auth0.com/blog/2015/12/11/get-your-twitter-share-count-back-with-a-webtask/).

You can start experimenting with **webtasks** by creating a free Auth0 account below, or head directly to [webtask.io](https://webtask.io).

### 7 - Faultless Database Migration

**Auth0** utilizes a built-in, enterprise-class, highly scalable and available database that is ideal for keeping track of millions of users. The first time a user or device logs in to Auth0, they will not have a record in the built-in Auth0 database, so Auth0 will use its connection to the existing external user database to get the record, including any user data that is to be migrated into the Auth0 built-in database. As well as completing the authentication request, Auth0 adds the newly acquired user record to its built-in database. Over the course of a few weeks or months, a majority of the users will have been automatically migrated over without noticing anything has changed. The rest of the records can then be bulk imported into Auth0 at any time but they will require password resets. Once the process is complete the existing external database can be retired. For more details, see [https://auth0.com/docs/connections/database/migrating](https://auth0.com/docs/connections/database/migrating)

<img src="https://cdn.auth0.com/content/email-wall/use-cases/database-migration/database-migration-logic.png" alt="Database Migration Logic" />

_Logic used to migrate users to Auth0 database_

<img src="https://cdn.auth0.com/content/email-wall/use-cases/database-migration/data-migration-block-diagram.png" alt="Migration on the fly to Auth0 database"/>

_User identities in an existing external database are migrated on the fly to the built-in database_

With **Auth0**, migrating user data to the Auth0 database to meets your scale, availability, performance or security goals is easier than you think.

### 8 - Technical Support

**Auth0** provides [libraries](https://github.com/auth0/) and [docs](https://auth0.com/docs) that enables developers to set up Authentication and Authorization in their apps within minutes. We support client-platforms: Vanilla JS, React, EmberJS and AngularJS, server-platforms: PHP, Nodejs, Ruby, Golang, Java, Python, .NET, Laravel, Wordpress, Joomla, Sharepoint and mobile-platforms: Android, iOs-objc.

**Auth0** is equipped with a great technical support team that's available to answer your questions and concerns about using any of the tools we provide.


## What people are saying about Auth0

"Auth0 was the only SaaS platform we looked at because it was clear that it was outstanding. Honestly" - David Bernick, Harvard Medical School

"We took a strategic bet on Auth0 by using their sophisticated identity infrastructure, rather than building it ourselves." - James Gardner, SVP of Product at Mindjet

"Integrating with Auth0 as a hub for customer SSO simplifies development, allowing our customer base to exponentially grow" - Cris Concepcion, Safari Books Online

"Auth0 blew our minds. The solution was implemented quickly, and infrastructure requirements were held to a minimum." - Amol Date, JetPrivilege

"With Auth0, we can plan and integrate identity architecture early, saving critical time and ensuring security" - Stephan Berard, Schneider Electric
