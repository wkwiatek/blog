# Understanding Angular 2 change detection

## Introduction to change detection
Angular 2 is somewhere near the final release. Although you've heard a lot of thinks about the components etc. the most valuable think in my opinion was the redesign of the core change detection system. As you remember digest loop from AngularJS - the performance was the issue. Now it's not.

### Why do we need it?
The question is - why do I have to bother? Generally the power of modern JavaScript frameworks works in similar manner; something changes in the model makes this change visible in the UI. That's when change detection comes in. Something have to trigger this propagation to the view. As mentioned before, in AngularJS we had a digest loops which checked every single reference that was set to be watched for value changes. When Angular found out everything's stable (no infinite loops etc.) then it propagated changes to the view. Although it was not efficient, it worked for a long time. The problem was also tracking asynchronous events. You also have used `$scope.$apply(...)`, haven't you? To understand why it was needed let's start with the beginning.

### How Javascript works
JavaScript runtime works on a single threaded engine, you know this. You've probably heard about stack (also from other programming languages). But that's everything what JS is about. Let's take the code below:
```js
console.log('Hey')
setTimeout(() => {
   console.log('Hello from timeout!')
}, 1000);
console.log('Hi')
```

You know that you'll see in a console:
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
every instruction will be put onto the stack and run one by one. There's no possibility you can see 3 before 2 or 1. You'll end up with:
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
- You have no way to ensure 1000ms is *exactly* 1000ms (it's at least 1000ms)

The have complete understand of event loop and what's going on in the browser I encourage you to take a look at Philip Roberts talk: https://www.youtube.com/watch?v=8aGhZQkoFbQ

### Zones
How does it compare to the Angular and change detection? Tracking to the object in synchronous code is fairly easy, however when it comes to the asynchronous one - it's not. That's AngularJS forced you to use `$scope.$apply(...)` each time asynchronous action was made. Or use angular way of doing asynchronous actions: `$timeout`, `$http` and so on. The thing is - if something was made outside of controller (even a perfectly valid change to the reference object) Angular didn't know about it so it didn't fired any event to reflect changes to the UI.

On the other hand we have Angular 2. It dropped all of this stuff connected to digest cycles and uses now *Zones*. Zones are able to track the context of asynchronous actions by monkey-patching them (overwriting them with it's own code) which then invoked the desired action but with some additional information attached. This additional information is the context. This way Angular will know in which component asynchronous action was invoked.

The big win of this approach is you can use browser API natively and Angular will know what is going on without forcing you to wrap it manually. The drawback is: Zone really overwrites asynchronous actions which is kinda hacky solution and may affect other (existing) code if your not relying only on Angular in the app.

But how exactly Angular is notified about the change? Angular uses own version of the Zone called `NgZone` which notifies about finished asynchronous action with `onTurnDone` event. Angular change detection waits for the event to perform change detection and check what need to be updated in the UI and what doesn't have to. That's the core behavior.

## Make use of change detection in the app
All described above is just about what's going under the hood. Equally important for us is how can we make use of it. Unlike AngularJS, Angular 2 gives the possibility to control the change detection. However, Angular team claims that even without any performance tweaking is 3-10 times faster than the previous one and for most apps is just fast enough. But it can much faster. Let's have an example.

[http://embed.plnkr.co/HR7ssEuPaWwlVKJPzZtJ/](http://embed.plnkr.co/HR7ssEuPaWwlVKJPzZtJ/)

There is very typical problem covered - rendering a list. So there is one component containing a list of another components that have some input data. So generally we have container with data and dumb component just for rendering single list item. Nothing fancy here but the getter and `ngOnChange`. What is done here? `ngOnChange` reacts on every input change and the getter adds additional logging each time rowData are fetched. Note we're not using it anywhere outside of the template. It means it's fired by the Angular itself. And what happens? We have single change on the input but there are hundreds of getter logs over there. Why is that? Angular is notified about the change from some component and have to check how it affected the current state so checks all the value for the change. Actually the team says it can make thousands of such checks in milliseconds but it's still a waste of time and it can harm our big data driven application.

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

The most important part here is the `const` declaration. What does it mean? Constant right? If `users` is constant how are we able to modify it? Well, that's how JavaScript works! The `const` prevent us from modifying a reference to the particular object in JavaScript. What `push` method of `Array` is doing is really appending another object to the existing array (with no reference change). Let's go to another very usual example:
```js
const user = {
  name: 'Max',
  age: 30
}

user.age = 31
```

The same thing applies. Although we can't modify whole object to be another one (reference change), we still can change some part of this object!

This is the problem why checks we discussed before are not so good. If you'd like to check if the object is the same as was before you have to *deeply check all of its properties*. And it's not efficient.

How can we force object to be new one with changed property? It's actually quite easy with new ECMAScript Object spread properties proposals:
```js
const user = {
  name: 'Max',
  age: 30
}

const modifiedUser = { ...user, age: 31 }
```

### Change Detection Strategies
The good part is: now we can say to the Angular that *we know what we're doing*. To modify change detection behavior we are able to use `ChangeDetectionStrategy` which generally has one very interesting value: `OnPush`. It makes component with this strategy applied looking at the values inside only when the reference on the input changes or some event has been fired from the component.  

So let's add the onPush strategy to our previous example:
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

You can try it on a plunker now and see the difference. The huge improvement is - there's now only one getter call for one change! We didn't nothing more as our input data are strings which are being changed so that reference on input changes. Reference for the rest of the components hasn't changed so Angular doesn't even look at it.

### App Structure
How can we build app to make most performant one? With Angular 2 is actually quite easy idea. As in all of the component frameworks nowadays you should have dumb and smart components. These dumb components which are meant to be only for displaying data from the input or handling user events are ideal volunteers for having the `OnPush` strategy. On the other hand smart components will sometimes require to watch for more things than the input and the events so be careful with setting the `OnPush` strategy everywhere.


## Conclusion
### Performance can increase a lot
### Eventually Angular can be tweaked
