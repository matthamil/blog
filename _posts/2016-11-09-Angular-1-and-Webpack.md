---
layout: post
title: Angular 1 and Webpack
---

Angular is a powerful front-end MVC framework. As I was building my front-end capstone at Nashville Software School (I wrote about it [here](http://matthamil.me/Machine-Learning-with_JavaScript/)), I wanted to integrate an `npm` package into my Angular 1 app.

In a vanilla Angular application using Bower as your dependency manager, you lose out on the opportunity to include a lot of fantastic libraries into your project. There are plenty of `npm` modules that could very well work on the front-end but simply can't be used due to the browser's lack of ES6 `import` support. If you are already writing your Angular 1 code in ES6 and using a build tool like Grunt or Gulp to compile to ES5--or if you aren't and want to get started writing in ES6--I strongly recommend using Webpack.

## What is Webpack?

[Webpack](https://webpack.github.io/) is a module bundler with super powers. You can do everything you can do with Grunt/Gulp and import npm modules into your front-end code!

> _I can already use Browserify and Gulp/Grunt to do that, can't I?_

Yes, however, Browserify relies on using the `require()` and is not aware of your entire build process. You can even forget about `bower_components` entirely with Webpack and manage all of your dependencies with `npm`!

Let's get started transforming a very basic Angular 1 application into a Webpack-friendly app!

---

## Our Original Angular 1 App

### Project Hierarchy

```
 - app/
     - controllers/
         - MyController.js
     - factories/
         - MyFactory.js
     - app.js
 - partials/
     - myPartial.html
 - css/
 - lib/
     - bower_components/
 - index.html
```

#### MyController.js
```javascript
app.controller('MyCtrl', function($scope, MyFactory) {
  $scope.message = MyFactory.getMessage();
});
```

#### MyFactory.js
```javascript
app.factory('MyFactory', () => {
  function getMessage() {
    return 'Hello Angular!';
  }

  return {
    getMessage
  };
});
```

#### app.js

```javascript
var app = angular.module('MyApp');
```

#### index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Angular</title>
  </head>
  <body ng-app="MyApp">
    <div ng-controller="MyController">
      {{message}}
    </div>

    <!-- Angular Dependencies -->
    <script src="lib/bower_components/angular/angular.min.js"></script>

    <!-- App Files -->
    <script src="app/app.js"></script>

    <!-- Factory -->
    <script src="app/factories/MyFactory.js"></script>

    <!-- Controllers -->
    <script src="app/controllers/MyCtrl.js"></script>
  </body>
</html>
```

This is our basic Angular 1 app. We have one controller, one factory, and one script file to build our Angular application. As this app grows, the number of script tags also grows. Yikes.

Let's start to configure this to work with Webpack!

## Webpack Setup

In order to get Webpack up and running, we first have to do some configuration to tell Webpack what exactly it will be doing to our code. The syntax might look outlandish at first, but trust me, it will make more sense once you start toying around with it.

In the root of our project, let's create a `webpack.config.js` file.

#### webpack.config.js

```javascript
import webpack from 'webpack';

module.exports = {
    // This creates a source map to make your
    // console errors a lot more developer friendly
    devtool: 'inline-source-map',
    // Webpack will start doing its thing here
    entry: {
        app: './app.js',
        vendor: ['angular']  
    },
    // Where Webpack spits the output
    output: {
        path: __dirname + '/dist',
        filename: 'bundle.js'
    },
    module: {
      // These are Webpack plugins (commonly referred
      // to as "loaders" in the Webpack community).
      // Babel will compile our ES6 to ES5
      loaders: [
        {
          test: /\.js$/,
          exclude: /node_modules/,
          loader: 'babel-loader'
        }
      ]
    }
};
```

Great! Now to be able to use Webpack, we need to install it with `npm`. Let's install a few dependencies we'll need to run Webpack.

We need to install Webpack globally to run it.

> ```Bash
npm i -g webpack
```

Now we need to install all of our dependencies to compile ES6 to ES5 using Babel.

> ```Bash
npm i --save-dev webpack babel-core babel-loader babel-preset-es2015
```

We'll need a `.babelrc` file in the root of our directory to configure Babel.

#### .babelrc

```json
{
  "presets": ["es2015"]
}
```

Webpack will now take all of our Javascript inside of our `app/` directory and bundle it accordingly. It is going to create a directory called `dist` at the root of our project. The `bundle.js` file created inside `dist` will contain our bundled Javascript. Now we need to point the `<script>` tag in `index.html` to `dist/bundle.js` and remove all other `<script>` tags containing parts of our Angular app!

## Modularizing our Angular App

#### index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Angular</title>
  </head>
  <body ng-app="MyApp">
    <div ng-controller="MyController">
      {{message}}
    </div>

    <!-- Bundled Javascript -->
    <script src="dist/bundle.js"></script>
  </body>
</html>
```

Doesn't that look a lot cleaner already? You can forget about updating your `<script>` tags every time you rename a file or add a new one!

Since Webpack relies on modules to know how to bundle your code, we'll need an Angular module to be bundled into our app code. Angular has an `npm` package that we can use (instead of using `bower`).

```Bash
npm i angular --save
```

Let's work from the ground up to convert our Angular code into modules. First, let's convert our controller into a module.

```js
function MyController($scope, MyFactory) {
  $scope.message = MyFactory.getMessage();
}

export default MyCtrl;
```

Woah, now we're writing a _plain Javascript function!_ We're also exporting it to be used elsewhere in our app. We're going to create a function that takes all of our controller functions and creates actual Angular controllers with them. Inside your `Controllers` folder, create an `index.js` file.

#### Controllers/index.js

```js
import angular from 'angular';
import MyController from './MyController';

angular.module('MyApp').controller('MyController', MyController);

// Add other controllers like so:
// angular.module('MyApp').controller('LoginCtrl', LoginCtrl);
// Don't forget to add new import statements
// at the top for each new Controller
```

Our `Controllers/index.js` file is our centralized location in our app that instantiates our controllers. We'll do the same for our factories.

#### Factories/MyFactory.js

```js
function MyFactory() {
  function getMessage() {
    return 'Hello Angular!';
  }

  return {
    getMessage
  };
}

export default MyFactory;
```

Now our factory functions _are plain Javascript functions too!_ If we wanted to give any part of our code access to an npm package , we can use the `import` syntax at the top of our file. Webpack will know to grab the code from our `node_modules` folder and make it available for you.

#### Example: Using an npm module inside our Angular app

```js
// Example import in a Factory
// Let's say we did an npm install firebase
// and wanted to use it in our project
import Firebase from 'firebase';

function FirebaseFactory() {
  Firebase.doThing(); // Magical importing ✨
  ...
  return { ... };
}

export default FirebaseFactory;
```

Let's create a similar `index.js` file like we did before for our Controllers, except this time it's for our Factories.

#### Factories/index.js
```js
import angular from 'angular';
import MyFactory from './MyFactory';

angular.module('MyApp').factory('MyFactory', MyFactory);
```

Okay, we've set up our factories and our controllers to be ready to be bundled! We still have to refactor our `app/app.js` file to bring all of the magic together.

#### app/app.js

```js
import angular from 'angular';

// If you have dependencies for Angular app
// like angular-route, you can import
// them like so
// import 'angular-route';

const app = angular.module('MyApp', [ /* dependencies */ ]);

// Loading Controllers
import './controllers';
// Loading Factories
import './factories';
```

Congratulations! You converted an Angular 1 app into an Angular 1 app with Webpack! You can now search through `npm` for _any_ dependencies you might want to add to your app. Keep in mind that some dependencies are only meant for Node, such as Express.

## The Final File Structure

```
 - app/
     - controllers/
         - MyController.js
         - index.js
     - factories/
         - MyFactory.js
         - index.js
     - app.js
 - partials/
     - myPartial.html
 - css/
 - node_modules/
 - dist/
 - index.html
```

## Running Webpack

In your `project.json`, let's add a script to the `scripts` section to enable us to run `npm build` to build our application with Webpack.

#### project.json

```json
{
  ...
  "scripts": {
    "build": "webpack"
  }
  ...
}
```

Webpack also has a great [dev server plugin](https://webpack.github.io/docs/webpack-dev-server.html) that isn't hard to configure too. It will rerun `webpack` any time it detects changes in your project.

You did it! Now go build something great with your modularized Angular 1 app!

Tweet me your awesome apps you're building with Angular and Webpack!

[@_matthamil](http://www.twitter.com/_matthamil)
