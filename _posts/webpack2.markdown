---
layout: post
title: "Building JavaScript with Webpack 2"
description: "Webpack 2 comes with support for ES2015 modules and tree-shaking. We'll see how to build a solid config for development and production."
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

## State of the Art Build Systems
Over the last few years JavaScript apps evolved from add-ons to dynamically validate forms to fully featured apps living in the browsers. As As app size increased, the community started to think of ways to modularize JS code. This led us to [browserify](http://browserify.org), which took the **NodeJS** module system to the web.

On the other hand we've seen a growing number of automation tools like [Grunt](http://gruntjs.com) or [gulp](http://gulpjs.com) that were to help us manage repeating tasks e.g. minifying code. Webpack combines the best of both worlds, and with npm scripts, is about to be the only solution you need to develop and build modern JavaScript application.

## What the Webpack Really Is
If you take a look at the brand new [webpack website](https://webpack.js.org), you'll see a nice graphics saying it bundles everything into static assets. It may seem like a simple thing, but in fact, webpack is much more than just a bundler. Due to the loaders it can import different files in specific way, and you can also add some plugins which are not involved directly during importing but may help with some generic tasks like copying or minifying. That makes webpack (with just a little help of npm) the only tool you'll probably need to automate tasks.

## Improvements Over Webpack 1
**Webpack 2** is a solid improvement over the first version, but the concepts remain the same - it's bundler on steroids. The major change is *support for ES2015 modules*. This also means splitting code into chunks, and then dynamically loading it on demand using `import` method. There is also a tree-shaking algorithm introduced. (You can read more about it here: [http://www.2ality.com/2015/12/webpack-tree-shaking.html](http://www.2ality.com/2015/12/webpack-tree-shaking.html)). Tree shaking allows **Webpack 2** to remove unused code. You can look at the gist containing all of the changes too: [https://gist.github.com/sokra/27b24881210b56bbaff7](https://gist.github.com/sokra/27b24881210b56bbaff7).

## Configuring Webpack 2 for Development
### Initialize project
First we're going to initialize an empty project with the `npm init` command. Our goal here will be to have two directories: `src` and `dist`. Inside the first we'll have the source code of our application. In the `dist` we'll have a bundle generated from the source directory.

To start, we have to create our entry points for the application: `index.html` and `index.js`. In `index.html` we'll add following content:

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

There's really nothing here but the script tag. Note the source name, it's called `bundle.js`, not `index.js`. This is not a mistake. What we're going to do is to create the `bundle.js` file using *webpack* and `index.js` as entry point.

To do this we need to install a *webpack* itself. We can install it using the following command:
```
npm i webpack --save-dev
```

Now we can create a *webpack config file*. It's called `webpack.config.js` by default and it's placed in the root directory. Basically, it's a NodeJS file that exports either a config object or a function that returns a config object. We'll go with the first, easier version right now. The very basic webpack config has to include an entry point, and the name of the output:
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

 Now, let's add some code to the `./src/index.js` file and we're almost there:
```js
console.log('Hello webpack!')
```

All we have to do now is run our app. To achieve that we will need a server, luckily *webpack* has it's own server for the development process which is called `webpack-dev-server`. We can install it now:
```
npm i webpack-dev-server --save-dev
```

After the `npm install` command is finished, the webpack dev server is installed in `node_modules/webpack-dev-server` directory. Runnable files from packages are also pushed to the `node_modules/.bin` directory. It means, we don't have to install packages globally to use it. We can do as follows:
```
./node_modules/.bin/webpack-dev-server
```

And now the `webpack-dev-server` command will be run. But we are still missing one thing. Our working directory will not be the root but `src`. We need to add additional parameter to the script:
```
./node_modules/.bin/webpack-dev-server --content-base ./src
```

Additionally you can append `--open` and the app will be opened automatically.

### npm scripts
Instead of remembering and typing the whole command each time, we can use *npm* scripts to make it easier to manage. In `package.json` there's a special property called `scripts` which is the place where you can put all commands that are required to run your app, and give tham a name. Let's put the previous command as a `dev` *npm* script:
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
Now it's time to extend our app a little bit. First, instead of printing something to the console, let's add some html, and populate it using JavaScript. In `index.html` we'll now add custom tag:
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

The main purpose of *webpack* is to manage modules. What we're going to do now is import some file using ES2015 import syntax. Let's add another file called `app.js`:
```js
const renderApp = element => {
  element.innerText = 'Hello webpack!'
}

export default renderApp
```

This file is exporting the function which expects an element to be passed into it. In `index.js` we can import the file and then use it in the following way:
```js
import renderApp from './app'

renderApp(document.getElementsByTagName('my-app')[0])
```

We now see exactly the same visual output, but our code is now split into more files, and it wasn't that hard! As it may seem not so valuable at the moment, it'll give a productivity boost when the application grows up.

## Configuring Webpack 2 for Production
We now have a working app but there are a few problems with it:
- not every browser understands ES2015 so it'd be cool to have a `babel`
- there's no minification
- we're using a development server

### Separating Configs
What we're going to do next, is create more *webpack* config files. The goal is to have:
- `webpack.base.config.js`
- `webpack.dev.config.js`
- `webpack.prod.config.js`

The `dev` and `prod` configs will be there to contain the additional stuff needed over the `base`. [webpack-merge](https://github.com/survivejs/webpack-merge) will be a helpful little library:
```
npm i webpack-merge --save-dev
```

Now our `webpack.config.js` becomes `webpack.base.config.js` and we're creating the additonal file called `webpack.dev.config.js`:
```js
const webpackBaseConfig = require('./webpack.base.config.js')
const merge = require('webpack-merge')

module.exports = merge(webpackBaseConfig, {
  devtool: 'source-map'
})
```

To differentiate the builds we've added `devtool: 'source-map'` property for dev mode. Since we changed the config filename, we have to reflect that change in our *npm* scripts too. Update the `dev` command to look like:
```
  "dev": "webpack-dev-server --content-base ./src --open --config ./webpack.dev.config.js"
```

Now you can go to the app and turn on the developer's console to see the `bundle.js` file and its source maps.

### Building real file
Although we haven't changed anything from the end users perspective (the app is still printing the same text), we introduced quite a solid structure for developer's mode. Now we want to be production ready too.

First, we need to get rid of the cool `webpack-dev-server`. It keeps all of the source files in memory, which is a huge advantage while developing, but now we need a single file to be put on the server. To make it happen let's create a new file called `webpack.prod.config.js` and populate it with very similar content to the `webpack.dev.config.js`:
```js
const webpackBaseConfig = require('./webpack.base.config.js')
const merge = require('webpack-merge')

module.exports = merge(webpackBaseConfig, {})
```

For now we'll leave the config the same as the base. Next, let's add a new *npm* script for this build:
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
Note that it's the only file that was generated. By default *webpack* is not creating `index.html` file so we need to take care of it by ourselves. Keep in mind that we put the `index.html` in the `src` directory, so all we need is to really copy that. Thanks to the *webpack* community we have a lot of plugins that we can use like [copy-webpack-plugin](https://github.com/kevlened/copy-webpack-plugin):
```
npm i copy-webpack-plugin --save-dev
```

Plugins have a special place in *webpack* config and their purpose is to do things that are not possible using the `loaders` (which we'll cover in a second). To apply a plugin we have to go the docs of the particular plugin. In case of `copy-webpack-plugin` we can do the following inside `webpack.prod.config.js`:
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
The power of *webpack* is the ability to process files before making a bundle. General idea is to have loaders created by the community which preprocess files which are about to be imported. As an example we'll take the [babel-loader](https://github.com/babel/babel-loader). To install it we need a few packages:
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

Last thing to have a fully working production build is to add minification. We can achieve this through the *UglifyJS* plugin. This plugin is built into *webpack* itself, so we can go and use it in the similar way we did with copy plugin:
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

## Final Thoughts
No matter whether you use **Angular**, **React**, or **Vue.js** - **webpack** is here to stay. It's a very powerful tool which makes development easier without introducing too much overhead. The ease of configuration is a big plus here. However, it doesn't make it simple under the hood. With proper loaders, and with npm scripts, it can cover all the automation tasks that **Grunt** or **gulp** were used for.

