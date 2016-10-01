---
layout: post
title: "Making use of RxJS in Angular 2"
description: "Angular 2 is built on the top of RxJS. It can be helpful for the app too."
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
- rxjs
- functional
- async
related:
-
-
-

---

---

**TL;DR** Angular 2 incorporates RxJS and uses it internally. We can make use of some RxJS goodies and introduce FRP to write more robust code.

---

Recently on Auth0 blog you could see the introduction to RxJS which I strongly recommend before taking this one. You can go directly to the [article](https://auth0.com/blog/understanding-reactive-programming-and-rxjs/) and then go back!

## Introduction
RxJS is all about streams, operators to modify them and observables.

### Functional Reactive Programming
FRP have recently became a buzzword. To give you a deeper understanding on that topic there is awesome post from Andre Stalz - [The introduction to Reactive Programming you've been missing](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754). What you should know from this comprehensive post? Reactive programming is actually programming with asynchronous data streams. Like RxJS streams. But where comes the word *functional*? Functional is about how we can modify these streams and create another. Stream can be used as an input to another stream. We have bunch of operators in RxJS to do things like this. So, can we do some of FRP with RxJS? The short answer is: yes! And we'll do so with Angular 2.

## Angular 2 and RxJS
When we want to start playing with RxJS in Angular, all we need to do is add operators we want to use. The RxJS itself is Angular dependency so it's just ready to use.

### Passing observables to the view
We are about to start with some observables created ad hoc. Let's create an observable from the JavaScript array:

```ts
const items = Observable.of([1, 2, 3])
```

Now we can use created observable as component's property and pass it into the view. Angular 2 introduced new filter which will be a perfect fit here. It's called `async`. Its purpose is to unwrap promises and observables. In case of observable it'll pass last value of the observable:


```ts
import { Component } from '@angular/core'
import { Observable } from 'rxjs/Rx'

@Component({
  selector: 'my-app',
  template: `
    <ul>
      <li *ngFor="let item of items | async">
        {{ item }}
      </li>
    </ul>
  `
})
export class AppComponent {
  public items = Observable.of([1, 2, 3])
}
```

We should see a list of elements in the browser.

This is our *hello world* example to see how the `async` works and what can we use it for.

### Http
Angular 2 relies on RxJS in some of the internal features. One of most known is the *Http* service. In AngularJS it was a promise-based service, now it's an observable-based one. It means we can also make use of async pipe here. Let's try to make real-world example with service now. What we want to achieve is to fetch list of repose for the Auth0 user:

```ts
import { Injectable } from '@angular/core'
import { Http } from '@angular/http'
import { Observable } from 'rxjs/Rx'
import 'rxjs/add/operator/map'

@Injectable()
export class RepoService {
  constructor(private _http: Http) {}

  getReposForUser(user: string): Observable<any> {
    return this._http
      .get(`https://api.github.com/users/${user}/repos`)
      .map((res: any) => res.json())
  }
}
```

Here we have the service which expose `getReposForUser` method to make an http call. Note the return type of the method - it's an `Observable<any>`. Now we can add it into the module and use it in the component:

```ts
@Component({
  selector: 'my-app',
  template: `

  `,
})
export class AppComponent {
  public repos: Observable<any>

  constructor(repoService: RepoService) {
    this.repos = repoService.getReposForUser('auth0')
    console.log(this.repos)
  }
}
```

Now something important has just happened. You can take a look into the *Network* tab of your developer tools in the browser. No call was made. But let's add the for loop with async pipe:

```ts
@Component({
  selector: 'my-app',
  template: `
    <ul>
      <li *ngFor="let repo of repos | async">
        {{ repo.name }}
      </li>
    </ul>
  `,
})
export class AppComponent {
  public repos: Observable<any>

  constructor(repoService: RepoService) {
    this.repos = repoService.getReposForUser('auth0')
  }
}
```

Now it's fired and prints the repos correctly. Why is that?

#### Hot and cold observables
Above example shows the observable which is an `http.get` is **cold**. What does it mean? Each subscriber sees the same events from the beginning as it would just start. It's independent of any other subscriber. It also means if there's no subscriber, no value is emitted! See this one in action:

```ts
export class AppComponent {
  public repos: Observable<any>

  constructor(repoService: RepoService) {
    this.repos = repoService.getReposForUser('auth0')
    this.repos.subscribe()
    this.repos.subscribe()
  }
}
```

Now you'll be able to see 3 calls. You can now see one more thing - `async` makes a subscription under the hood.

On the other hand we have **hot** observables. The difference is, no matter what is the subscribers number, it's fired just once. And we can make our observable hot instead of cold by using `share` operator:

```ts
// ...
import 'rxjs/add/operator/share'

@Injectable()
export class RepoService {
  constructor(private http: Http) {}

  getReposForUser(user: string): Observable<any> {
    return this.http
      .get(`https://api.github.com/users/${user}/repos`)
      .map((res: any) => res.json())
      .share()
  }
}
```

Now you should see just one call.

### Handling events
We've just covered the case you've probably used RxJS observables (even if you were not aware of that). But there are more things to do with streams even if Angular doesn't require you to do so. Now we move on to the *on click* events. The traditional, imperative way of handling click events in Angular 2 is as follows:

```ts
@Component({
  selector: 'my-app',
  template: `
    <button (click)="handleButtonClick(1)">
      Up Vote
    </button>
  `,
})
export class AppComponent {
  handleButtonClick(value: number) {
    console.log(value)
  }
}
```

But we can create a stream of click events using RxJS `Subject`. *Subject* is observer and observable at once. It means it can emit value (using `.next()`) and you can subscribe to it (using `subscribe`).

Here you can see the same case achieved with functional approach using RxJS:

```ts
@Component({
  selector: 'my-app',
  template: `
    <button (click)="counter$.next(1)">
      Up Vote
    </button>
  `,
})
export class AppComponent {
  public counter$: Observable<number> = new Subject<number>()

  constructor() {
    this.counter$.subscribe(console.log.bind(console))
  }
}
```

It's not such much different though. But let's try to add some more logic there. Like making sum of clicks and printing some text instead of just number.

```ts
@Component({
  selector: 'my-app',
  template: `
    <button (click)="counter$.next(1)">
      Up Vote
    </button>
  `,
})
export class AppComponent {
  public counter$: Observable<string> = new Subject<number>()
    .scan((acc: number, current: number): number => acc + current)
    .map((value: number): string => `Sum of clicks: ${value}`)

  constructor() {
    this.counter$.subscribe(console.log.bind(console))
  }
}
```

What's the most important here, *we define how that clicks stream will behave while declaring the variable*. We say that we don't really need clicks but only sum of them with some prepended text. It's our stream, not the pure click events. And we subscribe to the stream of summed values, not the original ones. In other words, *the key of functional programming is to make code declarative, not imperative*

### Comunication between components
There is one more thing there's worth to spend some time on. It's actually about dumb components in RxJS approach of Angular world. Last time I described the [change detection of Angular 2]() and what can we with it to fine-tune the app. Actually, what we'll just add to our code is the component which will use our `clicks$` stream as the input.

```ts
import { ChangeDetectionStrategy, Component, Input } from '@angular/core'

@Component({
  selector: 'my-score',
  template: 'Summary: {{ score }}',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class ScoreComponent {
  @Input() public score: number
}
```

Note that the component has `ChangeDetectionStrategy.OnPush` turned on so it means we assume the new reference will come as the input. Also there's nothing about streams here in the component. Just number as the input. How to make it happen on the other side? There's the `async` pipe once again:

```ts
@Component({
  selector: 'my-app',
  template: `
    <button (click)="counter$.next(1)">
      Up Vote
    </button>
    <my-score [score]="counter$ | async"></my-score>
  `,
})
export class AppComponent {
  public counter$: Observable<number> = new Subject<number>()
    .scan((acc: number, current: number): number => acc + current)
}
```

## Conclusion
### Maintainability
### Is it worth?

