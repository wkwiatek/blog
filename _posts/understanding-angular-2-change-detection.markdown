---
layout: post
title: "Understanding Angular 2 change detection"
description: "Angular 2 introduced new change detection system which makes apps far faster. Eventually the performance can be tweaked."
date: 2016-09-14 10:00
author:
  name: "Wojciech Kwiatek"
  url: "https://twitter.com/WojciechKwiatek"
  mail: "wojtek.kwiatek@gmail.com"
  avatar: "https://en.gravatar.com/userimage/102277541/a28d70be6ae2b9389db9ad815cab510e.png?size=200"
design:
  bg_color: "#171717"
  image: ""
tags:
- angular2
- angularjs
- change detection
- performance
- immutable
related:
-
-
-

---

---

**TL;DR** Angular 2 introduces reinvented change detection system which drops off digest cycles in favor of one way flow. Additionally it can be controlled now to take most of the framework's performance.

---

## Introduction to change detection
Angular 2 is somewhere near the final release. You've probably heard of some important changes due to the major version bump - its completely rewritten core, TypeScript as the language of choice, reinvented forms, RxJS, completely new router etc. But the most valuable think in my opinion was the redesign of the core change detection system. As you remember digest loop from AngularJS aka Angular 1 - the performance was the issue. Now it's not.

### Why do we need it?
The question is - why do I have to bother? Generally the power of modern JavaScript frameworks works in similar manner; something changes in the model makes this change visible in the UI. That's when change detection comes in. Something have to trigger this propagation to the view. As mentioned before, in Angular 1 we had a digest loops which checked every single reference that was set to be watched for value changes. When Angular found out everything's stable (no infinite loops etc.) then it propagated changes to the view. Although it was not efficient, it worked for a long time. The problem was also tracking asynchronous events. You also probably have used `$scope.$apply(...)` if you touched Angular 1 before. To understand why it was needed let's start with the beginning.

### How Javascript works
JavaScript runtime works on a single threaded engine. You've probably heard about stack (also from other programming languages). But that's everything what JS is about. Let's take the code below:
```js
console.log('Hey')
setTimeout(() => {
   console.log('Hello from timeout!')
}, 1000);
console.log('Hi')
```

We'll see in a console:
```
Hey
Hi
Hello from timeout!
```

And moreover, nothing is blocked during the 1 second wait period. So how the JS engine would do this with a single thread?

#### Synchronous code
Let's go step by step. If you have code like this:
```js
console.log('1')
console.log('2')
console.log('3')
```
every instruction will be put onto the stack and run one by one. There's no possibility you can see 3 before 2 or 1. We'll end up with:
```
1
2
3
```

Every time. Everywhere.

#### Asynchronous code
But let's go back to the timeout:
```js
console.log('1')
setTimeout(() => {
  console.log('2')
}, 0)
console.log('3')
```

What happens now? On stack we'll have:
```js
console.log
setTimeout
console.log
```

The trick here is how `setTimeout` works and what it really is. Yes, it will be invoked as normal synchronous action but all JS engine does is to give the steering to something else. There's also a bunch of browser APIs which are not a part of this single threaded process. There's a thing called event loop. This event loop goes one by one for the stack instructions and if it's empty then goes to the *callback queue*. And the reference to the `setTimeout` code is there. Once callback is done - the code will go the stack.

What does it mean? Two things:
- Everything what's inside asynchronous callback (as in `setTimeout`) will be run *after* any other synchronous code; this is why hacks like setTimeout(() => {}, 0) work
- We have no way to ensure 1000ms is *exactly* 1000ms (it's at least 1000ms)

The have complete understand of event loop and what's going on in the browser I encourage you to take a look at Philip Roberts talk: https://www.youtube.com/watch?v=8aGhZQkoFbQ

### Zones
How does all of this is related to the Angular and change detection? Tracking to the object in synchronous code is fairly easy, however when it comes to the asynchronous one - it's not. That's Angular 1 forced us to use `$scope.$apply(...)` each time asynchronous action was made. Or use angular way of doing asynchronous actions: `$timeout`, `$http` and so on. The thing is - if something was made outside of controller (even a perfectly valid change to the reference object) Angular didn't know about it so it didn't fired any event to reflect changes to the UI.

On the other hand we have Angular 2. It dropped all of this stuff connected to digest cycles and uses now *Zones*. Zones are able to track the context of asynchronous actions by monkey-patching them (overwriting them with it's own code) which then invoked the desired action but with some additional information attached. This additional information is the context. This way Angular will know in which component asynchronous action was invoked.

The big win of this approach is we can use browser API natively and Angular will know what is going on without forcing us to wrap it manually. The drawback is: Zone really overwrites asynchronous actions which is kinda hacky solution and may affect other (existing) code if we're not relying only on Angular in the app.

But how exactly Angular is notified about the change? Angular uses own version of the Zone called `NgZone` which notifies about finished asynchronous action with `onTurnDone` event. Angular change detection waits for the event to perform change detection and check what need to be updated in the UI and what doesn't have to. That's the core behavior.

## Make use of change detection in the app
All described above is just about what's going under the hood. Equally important for us is how can we make use of it. Unlike Angular 1, Angular 2 gives the possibility to control the change detection. However, Angular team claims that even without any performance tweaking is 3-10 times faster than the previous one and for most apps is just fast enough. But it can much faster. Let's have an example.

[http://embed.plnkr.co/HR7ssEuPaWwlVKJPzZtJ/](http://embed.plnkr.co/HR7ssEuPaWwlVKJPzZtJ/)

There is very typical problem covered - rendering a list. There is one component containing a list of another components that have some input data. Generally we have container with data and dumb component just for rendering single list item. Nothing fancy here but the getter and `ngOnChange`. What is done here? `ngOnChange` reacts on every input change and the getter adds additional logging each time rowData are fetched. Note we're not using it anywhere outside of the template.

It means the getter is fired by the Angular itself. And guess what happens? We have single change on the input but there are hundreds of getter logs over there.

Why is that?

Angular is notified about the change from some component and have to check how it affected the current state so checks all the value for the change. Actually the team says it can make thousands of such checks in milliseconds but it's still a waste of time and it can harm our big data driven application.

### Immutability
The cool thing about new change detection system is now we can tune it. Let's take a brake from Angular and consider following code:
```js
const users = [{
  name: 'John',
  age: 27
}, {
  name: 'Anna',
  age: 23
}]

users.push({
  name: 'Max',
  age: 30
})
```

The most important part here is the `const` declaration. If `users` is constant how are we able to modify it? Well, that's how JavaScript works! The `const` prevent us from modifying a reference to the particular object in JavaScript. What `push` method of `Array` is doing is really appending another object to the existing array (with no reference change). Let's go to another very usual example:
```js
const user = {
  name: 'Max',
  age: 30
}

user.age = 31
```

The same thing applies. Although we can't modify whole object to be another one (reference change), we still can change some part of this object!

This is the problem why checks we discussed before are not so good. If you'd like to check if the object is the same as was before you have to *deeply check all of its properties*. And it's not efficient.

How can we force object to be new one with changed property? It's actually quite easy with new [ECMAScript Object spread properties proposals](https://github.com/sebmarkbage/ecmascript-rest-spread):
```js
const user = {
  name: 'Max',
  age: 30
}

const modifiedUser = { ...user, age: 31 }
```

### Change Detection Strategies
The good part is: now we can say to the Angular that *we know what we're doing*. To modify change detection behavior we are able to use `ChangeDetectionStrategy` which generally has one very interesting value: `OnPush`. It makes component with this strategy applied looking at the values inside only when the reference on the input changes or some event has been fired from the component.  

Let's add the onPush strategy to our previous example:
```ts
import {ChangeDetectionStrategy, Component, Input} from '@angular/core';

@Component({
  selector: 'row',
  template: `
    <pre>{{ rowData }}</pre>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class RowComponent {
  ...
}
```

You can try it on a [plunker now](http://embed.plnkr.co/d9b07qginx7z9hGYyeME/) and see the difference. The huge improvement is - there's now only one getter call for one change! We didn't nothing more as our input data are strings which are being changed so that reference on input changes. Reference for the rest of the components hasn't changed so Angular doesn't even look at it.

### App Structure
How can we build app to make most performant one? With Angular 2 is actually quite easy idea. As in all of the component frameworks nowadays you should have dumb and smart components. These dumb components which are meant to be only for displaying data from the input or handling user events are ideal volunteers for having the `OnPush` strategy. On the other hand smart components will sometimes require to watch for more things than the input and the events so be careful with setting the `OnPush` strategy everywhere.

## Aside: Angular 2 Authentication with Auth0

Auth0 issues JSON Web Tokens on every login for your users. That means that you can have a solid identity infrastructure, including single sign-on, user management, support for social (Facebook, Github, Twitter, etc.), enterprise (Active Directory, LDAP, SAML, etc.) and your own database of users with just a few lines of code.

You can add Auth0 to an Angular 2 app really easily. There are just a few simple steps:

### Step 0: Sign Up for Auth0 and Configure

If you don't already have any Auth0 account, [sign up](https://auth0.com/signup) for one now to follow along with the other steps.

### Step 1: Add Auth0Lock to Your App

[Lock](https://auth0.com/lock) is the beautiful (and totally customizable) login box widget that comes with Auth0. The script for it can be brought in from a CDN link or with npm.

> Note: If you use npm to get Auth0Lock, you will need to include it in your build step.

```html
  <!-- src/client/index.html -->

  ...

  <!-- Auth0 Lock script -->
  <script src="https://cdn.auth0.com/js/lock-9.0.min.js"></script>

  <!-- Setting the right viewport -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />

  ...
```

### Step 2: Add an Authentication Service

It's best to set up an injectable service for authentication that can be used across the application.

With Auth0, we get access to the user's profile and JWT in the `lock.show` callback and these items can be saved in local storage for use later.

```js
// src/client/shared/auth.service.ts

import {Injectable, NgZone} from 'angular2/core';
import {Router} from 'angular2/router';
import {AuthHttp, tokenNotExpired} from 'angular2-jwt';

// Avoid name not found warnings
declare var Auth0Lock: any;

@Injectable()
export class AuthService {
  lock = new Auth0Lock('YOUR_AUTH0_CLIENT_ID', 'YOUR_AUTH0_DOMAIN');
  refreshSubscription: any;
  user: Object;
  zoneImpl: NgZone;

  constructor(private authHttp: AuthHttp, zone: NgZone, private router: Router) {
    this.zoneImpl = zone;
    this.user = JSON.parse(localStorage.getItem('profile'));

    // Add callback for lock `authenticated` event
    this.lock.on("authenticated", authResult => {
      self.lock.getProfile(authResult.idToken, (error, profile) => {

        if (error) {
          // handle error
          return;
        }

        // If authentication is successful, save the items
        // in local storage
        localStorage.setItem('profile', JSON.stringify(profile));
        localStorage.setItem('id_token', authResult.idToken);
        this.zoneImpl.run(() => self.user = profile);
      });
    });
  }

  public authenticated() {
    // Check if there's an unexpired JWT
    return tokenNotExpired();
  }

  public login() {
    // Show the Auth0 Lock widget
    this.lock.show();
  }

  public logout() {
    localStorage.removeItem('profile');
    localStorage.removeItem('id_token');
    this.zoneImpl.run(() => this.user = null);
    this.router.navigate(['Home']);
  }
}
```

### Step 3: Add a Click Handler to Login

To be able to use the "Login" and "Logout" methods, we need to inject the Auth service in the app.components.ts file.

```html
<!-- src/client/app.components.ts -->
import { Component } from '@angular/core';
import { AuthService } from './auth.service';

...
```

In your root NgModule, declare the service provider, as shown in the following code.

```js
<!-- src/client/app.module.ts -->

...
import { AuthService } from './auth.service';

@NgModule({
 imports: [
   BrowserModule,
   FormsModule,
   routing,
   HttpModule
 ],
 declarations: [
   AppComponent
 ],
 providers: [
   AUTH_PROVIDERS,
   AuthService
 ],
 bootstrap: [AppComponent]
})
export class AppModule { }
```

Now, we can use the methods from our authentication service in any of our components which means we can easily add a click handler to a "Login" and "Logout" button.

```html  
<!-- src/client/app.component.ts -->

  ...

  <button (click)="authService.login()" *ngIf="!authService.authenticated()">Log In</button>
  <button (click)="authService.logout()" *ngIf="authService.authenticated()">Log Out</button>

  ...
```

Once the user logs in, a [JSON Web Token](https://jwt.io/introduction) will be saved for them in local storage. This JWT can then be used to make authenticated HTTP requests to an API.

### Step 4: Make Authenticated HTTP Requests

With [**anuglar2-jwt**](https://github.com/auth0/angular2-jwt), we can automatically have our JWTs sent in HTTP requests. To do so, we need to inject and use `AuthHttp`.

```js
// src/client/ping/ping.component.ts

import {Component} from 'angular2/core';
import {Http} from 'angular2/http';

import {AuthHttp} from 'angular2-jwt';
import {Auth} from './auth.service';
import 'rxjs/add/operator/map';

@Component({
  selector: 'ping',
  template: `
    <h1>Send a Ping to the Server</h1>
    <button class="btn btn-primary" (click)="securedPing()" *ngIf="auth.authenticated()">Secured Ping</button>
    <h2>{{message}}</h2>
  `
})
export class Ping {
  API_URL: string = 'http://localhost:3001';
  message: string;

  constructor(private http: Http, private authHttp: AuthHttp, private auth: Auth) {}

  securedPing() {
    this.authHttp.get(`${this.API_URL}/secured/ping`)
      .map(res => res.json())
      .subscribe(
        data => this.message= data.text,
        error => this.message = error._body
      );
  }
}
```

### Step 5: Done!

That's all there is to it to add authentication to your Angular 2 app with Auth0!

## Conclusion

### Performance can increase a lot
One of the big advantages of using more strict change detection is the performance gain. Angular is meant to be used for rather big applications which can end up with handling a lot of dynamic data. What we really did there is taking the responsibility from the Angular to the programmer. By default every change should be reflected on the UI as Angular takes care of that but the price is the performance. Immutable or reactive code is harder to write but easier to maintain and reason about. The choice is your.

### Eventually Angular can be tweaked
The good is we eventually have a choice. In Angular 1 it was impossible to reach. The was some level you had to use React or other library to render UI instead of Angular templates as it was mainly too slow for greater amount of dynamic data. Now you have a complete solution with much more control over the internal behavior. However this one with combination of other changes made to Angular 2 makes the learning curve of the framework steeper.
