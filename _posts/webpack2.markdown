---
layout: post
title: "Building JavaScript with Webpack 2"
description: ""
date: 2017-02-01 10:00
author:
  name: "Wojciech Kwiatek"
  url: "https://twitter.com/WojciechKwiatek"
  mail: "wojtek.kwiatek@gmail.com"
  avatar: "https://en.gravatar.com/userimage/102277541/a28d70be6ae2b9389db9ad815cab510e.png?size=200"
design:
  bg_color: "#171717"
  image: ""
tags:
- webpack
- build
related:
-
-
-

---

---

**TL;DR** Webpack 2 is out. We will create simple, yet fully functional setup for both dev and production mode.

---

## Build systems state of the art
For the last few years JavaScript apps evolved from add-ons to dynamically validate forms to fully featured apps living in the browsers. As apps size was increasing, community started to think of a way of modularizing code in JS. This led us to the *browserify*, which took **NodeJS** module system to the web. On the other hand we've seen a grow of automation tools like *Grunt* or *gulp* that were to help us to manage repeating tasks e.g. minifying code, much easier. Webpack is somehow combining both worlds, and with npm scripts, is about to be the only solution you need to develop and build modern JavaScript application.


## Improvements over Webpack 1
**Webpack 2** is a solid improvement over the first version, but the concepts remain the same - it's bundler on steroids. The major change is *the support for ES2015 modules*. This means also splitting code into the chunks, and then dynamically load on demand using `import` method. There is also tree-shaking algorithm introduced. (You can read more about it here: [http://www.2ality.com/2015/12/webpack-tree-shaking.html](http://www.2ality.com/2015/12/webpack-tree-shaking.html)). Thanks to that, **webpack 2** is able to remove unused code. You can look at the gist containing all of the changes too: [https://gist.github.com/sokra/27b24881210b56bbaff7](https://gist.github.com/sokra/27b24881210b56bbaff7).

## Building for development
### Initialize project
First we're going to initialize empty project with `npm init` command. Our goal here will be to have two directories: `src` and `dist`. Inside the first we'll have the source code of our application. In the `dist` there will be a bundle generated from the source directory.

At the beginning we have to create our entry points for the application: `index.html` and `index.js`. In `index.html` we can put following content:

```html
<html>
  <head>
    <title>Webpack 2 Demo</title>
  </head>
  <body>

    <script src="bundle.js"></script>
  </body>
</html>
```

There's really nothing but the script tag. Note the source name, it's called `bundle.js`, not `index.js`. It's not a mistake. What we're going to do is to create the `bundle.js` file using the *webpack* and `index.js` as entry point.

To combine it together we need to install a *webpack* itself. We can install it using the following command:
```
npm i webpack --save-dev
```

Now we can create *webpack config file*. It's called the `webpack.config.js` by default and it's placed in the root directory. Basically, it's a NodeJS file which have export either config object or function which returns config object. We'll go with the first, easier version right now. The very basic webpack config have to include an entry point, and the name of the output:
```js
const path = require('path')

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: 'bundle.js'
  }
}

```

Note: `path` is NodeJS built-in library, and `path.resolve` is to get the absolute path of the file. `__dirname` is special NodeJS variable that contains the directory the program was run at.

Now you can something to your `./src/index.js` file and we're almost there:
```js
console.log('Hello webpack!')
```

All we have to do now is to run our app. To achieve that we will need some server, and *webpack* has it's own server for the development process which is called `webpack-dev-server`. We can install it now:
```
npm i webpack-dev-server --save-dev
```

After the `npm install` command it was installed to the `node_modules/webpack-dev-server` directory. Runnable files from packages are also pushed to the `node_modules/.bin` directory. It means, we don't have to install packages globally to use it. We can do as follows:
```
./node_modules/.bin/webpack-dev-server
```

And now the `webpack-dev-server` command will be run. But we still miss one thing. Our working directory will not be the root but `src`. We need to add additional parameter to the script:
```
./node_modules/.bin/webpack-dev-server --content-base ./src
```

Additionally you can `--open` too and the app will be opened automatically.

### npm scripts
Instead of remembering the whole command we can use *npm* scripts to make it easier to manage. In `package.json` there's a special property called `scripts` which is the place where you can put all commands that are required to run your app, and give tham a name. Let's put the previous command as a `dev` *npm* script:
```
{
  "scripts": {
    "dev": "webpack-dev-server --content-base ./src --open"
  },
  "devDependencies": {
    "webpack": "^2.2.0",
    "webpack-dev-server": "^2.2.0"
  }
}
```

Notice one difference, there's no `./node_modules/.bin/` in the command path. It's because *npm* is looking for a local scripts in this directory by default. Now you can run it by typing:
```
npm run dev
```

### Modules
Now it's time to extend our app a little bit. First, instead of printing something to the console, let's create some html there, and populate it using JavaScript. In `index.html` we'll now add custom tag:
```html
<html>
  <head>
    <title>Webpack 2 Demo</title>
  </head>
  <body>
    <my-app></my-app>

    <script src="bundle.js"></script>
  </body>
</html>
```

And in `index.js` you can add:
```js
const myAppElement = document.getElementsByTagName('my-app')[0]
myAppElement.innerText = 'Hello webpack!'
```

Now you should be able to see the text in the html which will save you some time while making next changes.

The main purpose of *webpack* is to manage modules. What we're going to do now is to import some file using ES2015 import syntax. Let's add another file called `app.js`:
```js
const renderApp = element => {
  element.innerText = 'Hello webpack!'
}

export default renderApp
```

This file is exporting the function which expects an element to be passed into that. In `index.js` we can import that file and then use it in a following way:
```js
import renderApp from './app'

renderApp(document.getElementsByTagName('my-app')[0])
```

We can see now exactly the same visual output, but our code is now split into the more files, and it wasn't that hard!

## Building for production
We do have a working app but there are few problems with it:
- not every browser understands ES2015 so it'd be cool to have a `babel`
- there's no minification there
- we're still using only development server

### Separated configs
What we're about to do now, is to create another *webpack* config files. The goal is to have:
- `webpack.base.config.js`
- `webpack.dev.config.js`
- `webpack.prod.config.js`

The `dev` and `prod` configs will be there to contain the additional stuff need over the `base` for these two types of build. [webpack-merge](https://github.com/survivejs/webpack-merge) will be helpful little library:
```
npm i webpack-merge --save-dev
```

Now our `webpack.config.js` becomes `webpack.base.config.js` and we're creating additonal file called `webpack.dev.config.js`:
```js
const webpackBaseConfig = require('./webpack.base.config.js')
const merge = require('webpack-merge')

module.exports = merge(webpackBaseConfig, {
  devtool: 'source-map'
})
```

To differentiate it somehow we've added `devtool: 'source-map'` property for dev mode. As we changed the name of the config while we have to reflect that changes in our *npm* scripts too. Update `dev` command will now look like:
```
  "dev": "webpack-dev-server --content-base ./src --open --config ./webpack.dev.config.js"
```

Now you can go to the app and turn on the developer's console to see the `bundle.js` file and its source maps at the end.

### Building real file
Although we haven't changed anything from the end user's perspective (the app is still printing the same text), we introduced quite solid structure for developer's mode. Now we wanted it to be production ready too.

First, we need to get rid of the cool `webpack-dev-server`. It keeps all of the source files in the memory, which is a huge advantage while developing, but now we need a single file to be put on the server. To make it happen we can now create new file called `webpack.prod.config.js` and populate it with very similar content to the `webpack.dev.config.js`:
```js
const webpackBaseConfig = require('./webpack.base.config.js')
const merge = require('webpack-merge')

module.exports = merge(webpackBaseConfig, {})
```

For now we'll live the config the same as the base. Now we can add some *npm* script to run the webpack in a build mode:
```
{
  "scripts": {
    "dev": "webpack-dev-server --content-base ./src --open --config ./webpack.dev.config.js",
    "build": "webpack --config ./webpack.prod.config.js"
  },
  ...
}
```

The command `npm run build` will create a real `bundle.js` file inside `dist` directory.
Note that it's the only file that was generated. By default *webpack* is not creating `index.html` file so we need to take care of it by ourselves. Keep in mind that we put the `index.html` in the `src` directory, so all we need is to really copy that. Thanks to the *webpack* community we have a lot of plugins so we can use [copy-webpack-plugin](https://github.com/kevlened/copy-webpack-plugin):
```
npm i copy-webpack-plugin --save-dev
```

Plugins have a special place in *webpack* config and their purpose is to do things that are not possible using the `loaders` (which we'll cover in a second). To apply some plugins we have to go the docs of the particular plugin. In case of `copy-webpack-plugin` we can do the following inside `webpack.prod.config.js`:
```
const webpackBaseConfig = require('./webpack.base.config.js')
const merge = require('webpack-merge')
const CopyWebpackPlugin = require('copy-webpack-plugin')

module.exports = merge(webpackBaseConfig, {
  plugins: [
    new CopyWebpackPlugin([
      { from: './src/index.html', to: 'index.html' }
    ])
  ]
})
```

Now we have both files in place. You can put files on any server you want, and it will just work. If you don't have anything up and running at the moment, you can give [http-server](https://github.com/indexzero/http-server) a try, and run `http-server dist` in the terminal.

The next step to make our JavaScript code production ready: we have to add *babel* and minification.

### Loaders
The power of *webpack* is the ability to process files before making a bundle. General idea is to have loaders created by the community which are preprocessing file which are about to be imported. As the example we'll take the [babel-loader](https://github.com/babel/babel-loader). To install it we need a few packages:
```
npm install babel-loader babel-core babel-preset-es2015 --save-dev
```

We can now go and use it with following config properties in `webpack.base.config.js`:
```
const path = require('path')

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: 'bundle.js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: 'babel-loader',
        exclude: /node_modules/,
        query: {
          presets: ['es2015']
        }
      },
    ]
  }
}
```

We had to add `module` property which then may have some `rules`. In `rules` we are defining the set of loader that have to be used when some particular filename is met. In our case it's a regular expression for all the files that are ending up with `.js`. Additionally we're adding the `query` which is a configuration of the loader. Now if you go and build your app with `npm run build` you'll see no ES2015 `const` etc.

Last thing to have fully working production build is to add a minification. We can achieve that through the *UglifyJS* plugin. This plugin is build into the *webpack* itself, so we can go and use it in the similar way we did with copy plugin:
```
const webpack = require('webpack')
const webpackBaseConfig = require('./webpack.base.config.js')
const merge = require('webpack-merge')
const CopyWebpackPlugin = require('copy-webpack-plugin')

module.exports = merge(webpackBaseConfig, {
  plugins: [
    new CopyWebpackPlugin([
      { from: './src/index.html', to: 'index.html' }
    ]),
    new webpack.optimize.UglifyJsPlugin({})
  ]
})
```

The build process should be complete now, and you should be able to see uglified `bundle.js` file.

## Final thoughts
No matter whether you use **Angular**, **React**, or **Vue.js** - **webpack** is here to stay. It's a very powerful tool which makes development easier without having too much overhead. The ease of configuration is a big plus here. However, it doesn't make it simple under the hood. With proper loaders, and with npm scripts, it can cover all the automation tasks that **Grunt** or **gulp** was used for.

