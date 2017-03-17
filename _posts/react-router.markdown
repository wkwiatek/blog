---
layout: post
title: "Declarative routing with React Router v4"
description: "React Router 4 was just released, and comes with brand new approach."
date: 2017-03-01 10:00
author:
  name: "Wojciech Kwiatek"
  url: "https://twitter.com/WojciechKwiatek"
  mail: "wojtek.kwiatek@gmail.com"
  avatar: "https://en.gravatar.com/userimage/102277541/a28d70be6ae2b9389db9ad815cab510e.png?size=200"
design:
  bg_color: "#171717"
  image: ""
tags:
- React
- React Router
related:
-
-
-

---

---

**TL;DR** React Router 4 was just released. It's here to solve some long-standing issues with completely new approach comparing to version 3.

---

## Problems with React Router 3
[React Router](https://github.com/ReactTraining/react-router) is often the first choice when you think of routing for a React application. It's not only the best known, but probably also the most comprehensive on the market currently. However, the way it works is not so obvious. React Router 3 (or below) is using component-like markup to provide route configuration. The `<Route> was not a React component - just a part of config. It means you can't simple wrap `<Route>` into higher order component and/or add some props. It turned out that the approach of static config produced more and more problems which you can see in a [v4 FAQ](https://github.com/ReactTraining/react-router/blob/v4/README.md#v4-faq).

Taking that into account, React Router team decided to drop the idea routes, and provide some React components instead. Now, we're going to take a look on what components do we have, and how we can use it.

## Starting New App
We will start with brand new app and will go through important pieces of React Router 4. To make it possible we'll use [create-react-app](https://github.com/facebookincubator/create-react-app) which is currently the easiest way to start solid, new application that uses React, and has the build system magic incorporated.

Let's start with installing `creat-react-app` globally:
```bash
npm install -g create-react-app
```

After finishing you'd be able to run:
```bash
create-react-app auth0-react-router-v4
```

This command will create all the files needed to develop and ship app into production environment. All we have to do is to go to the `auth0-react-router-v4` directory, and run:
```bash
npm start
```
Or `yarn start` if [Yarn](https://yarnpkg.com) is your package manager of choice.


## Introduction to React Router 4
First thing that we have to do to integrate router into our application is to install React Router. In version 4 of React Router it changed a little bit. Now packages are separted for different usage, and if we want to use React Router in the browser, we should install `react-router-dom`:
```bash
npm install react-router-dom --save
```

Now it's time to add first component which will act as the main page. We can go to `App.js` and import `BrowserRouter` and `Route` from `react-router-dom` package:
```jsx
import React, { Component } from 'react'
import { BrowserRouter, Route } from 'react-router-dom'

class App extends Component {
  render() {
    return (
      <BrowserRouter>
        <Route path="/" component={ () => <h1>Home view!</h1> } />
      </BrowserRouter>
    )
  }
}

export default App
```

The interesting parts here are:
- `BrowserRouter` - wrapper for all routes that are inside the application. It should be one of the top-level containers.
- `Router` - which takes `path` for specific `component` to be rendered.

> Note that in the situation above any url will match route. The `path` we provided was `/` which means that in url there should be at least `/` (which every single URL has), e.g. for `/home` it means `/home/*`. If we want to be strict we can add `exact` prop to the `<Route>`.

## Creating an App

### Very First Component

Ok, let's say we wanted to have an application which is an address book. It means we wanted to have some home page with a link to the address book, and the address book itself would contain:
- list of people
- currently chosen one

Something like this:
> Ask for some wireframe

What problems do we want to solve first?
- Home page with a link to address book
- list of addresses on the address book subpage (visible always when on /address-book)
- Some text when no address is chosen
- Details of the chosen address

Let's start with some components. Here we go with an improved home page component:
```jsx
const HomeComponent = () => (
  <div>
    <h1>Welcome!</h1>
    <Link to="/address-book">Go to the address book</Link>
  </div>
)
```

`Link` is a component from `react-router-dom` which is routing to proper `Route`. `to` attribute can be either an object or a string. We've used simpler version here as we don't need any query strings, additional hashes etc. When we have a link, we should have also a component that we're linking to. The path is `address-book` and we need a proper `Route`:
```jsx
<Route path="/address-book" component={ AddressBookHomeComponent }/>
```

And the component itself:
```jsx
const AddressBookHomeComponent = () => (
  <p>Chose some contact from the list</p>
)
```

Now, we should be able to see the following app:
![Router 0](auth0-router-0.gif)

### Adding More Routes

Okay, let's make it more complex. As per initial design we have a left column, which contains of a list of contacts, and right (main one) which shows currently chosen contact.

First, we need some data. As we don't want to increase complexity, it'll be a simple object:
```jsx
const contacts = [
  { id: 0, name: 'Adam', email: 'adam@auth0.com' },
  { id: 1, name: 'Ben', email: 'ben@auth0.com' },
  { id: 2, name: 'Claudia', email: 'claudia@auth0.com' }
]
```

Component to display above list can look like that:
```jsx
const AddressBookListComponent = () => (
  <div>
    <h2>All Contacts</h2>
    <ul>
      {
        contacts.map(contact => (
          <li key={ contact.id }>
            { contact.name }
          </li>
        ))
      }
    </ul>
  </div>
)
```

> Note: we're using `contacts` directly in the component. We did it just to not increase complexity with any of the state management libraries. However, you should not leave it as it is, and pass some stores through providers etc. You can read more about e.g. Redux [in another article](https://auth0.com/blog/secure-your-react-and-redux-app-with-jwt-authentication/).

To display that list we'll need another `Route`:
```jsx
<Route path="/address-book" component={ AddressBookListComponent }/>
```

If you look closely you'll be able to see that we just did the same `path` twice. Why we did that? For the same url we wanted to have both components to be render, and that's perfectly fine for React Router 4! We need some very basic styling to see the left, and right column:

```css
.app-container {
  display: flex;
}

.home-page {
  padding: 30px;
}

.address-book-list {
  flex: 1;
  padding: 10px;
  min-height: 100vh;
  background: #f4f4f4;
}

.address-book-contact {
  flex: 3;
  padding: 10px;
  background: #fff;
}
```

And our complete code will look like this:
```jsx
import React, { Component } from 'react';
import { BrowserRouter, Route, Link } from 'react-router-dom';

const contacts = [
  { id: 0, name: 'Adam', email: 'adam@auth0.com' },
  { id: 1, name: 'Ben', email: 'ben@auth0.com' },
  { id: 2, name: 'Claudia', email: 'claudia@auth0.com' }
]

const HomeComponent = () => (
  <div className="home-page">
    <h1>Welcome!</h1>
    <Link to="/address-book">Go to the address book</Link>
  </div>
)

const AddressBookListComponent = () => (
  <div className="address-book-list">
    <h2>All Contacts</h2>
    <ul>
      {
        contacts.map(contact => (
          <li key={ contact.id }>
            { contact.name }
          </li>
        ))
      }
    </ul>
  </div>
)

const AddressBookHomeComponent = () => (
  <div className="address-book-contact">
    <p>Chose some contact from the list</p>
  </div>
)

class App extends Component {
  render() {
    return (
      <BrowserRouter>
        <div className="app-container">
          <Route exact path="/" component={ HomeComponent } />
          <Route path="/address-book" component={ AddressBookListComponent }/>
          <Route path="/address-book" component={ AddressBookHomeComponent }/>
        </div>
      </BrowserRouter>
    );
  }
}

export default App;
```

And this is how it looks like:
![Router 1](auth0-router-1.gif)

### Dynamic Links

Everything starts to look like in the design, but we just meet first bigger problem: contacts on the left should be clickable, and should force right column to show a specific contact details.

To achieve this behavior we need to do two things to the routes:
- ensure that 'Chose some contact from the list' appears only when no contact is chosen
- create a `Router` for a contact with dynamic `id`

To get the first done all we need is to add `exact` to `AddressBookHomeComponent` `Route`:
```jsx
<Route exact path="/address-book" component={ AddressBookHomeComponent }/>
```

On the other hand, to get the second one done, we'll add another `Route`:
```jsx
<Route path="/address-book/:id" component={ ContactComponent }/>
```

Note the `:id` in the path. It means that part of the url after the `/address-book/` will be considered as variable, and we can get it inside the component. `ContactComponent` doesn't exist yet, so let's create it. From the React Router docs we can read that each component will get a `match` property, which we can use for our purpose:
```jsx
const ContactComponent = ({ match }) => (
  <div className="address-book-contact">
    <h3>Contact No. { match.params.id }</h3>
    <p>Email: { contacts[match.params.id].email }</p>
  </div>
)
```

Property [match](https://reacttraining.com/react-router/web/api/match) contains information about how a `Route`'s path matched given url. We can get params from this object, and look for our `id` name.

The last piece is to change left column list into list of links. We can use match for this one too to avoid repetitions:
```jsx
const AddressBookListComponent = ({ match }) => (
  <div className="address-book-list">
    <h2>All Contacts</h2>
    <ul>
      {
        contacts.map(contact => (
          <li key={ contact.id }>
            <Link to={ `${match.url}/${contact.id}` }>{ contact.name }</Link>
          </li>
        ))
      }
    </ul>
  </div>
```

Fully working version should look like the following:
![Router 2](auth0-router-2.gif)


