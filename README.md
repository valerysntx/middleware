## Stack

docker => `server.js` @ http:`3000` => babel/transform/proxy => [API server](#api-server) @ `localhost:3030`
  - `server.js` will proxify requests to `/api/*`
   
  - data fetching calls from the client go to `/api/*`.
  - static content from `/static`

## Why? 

- data hydration middleware
- ApiClient for both client / server
- one repo per one app => api + client
```
yarn why redux
```
* [React](https://github.com/facebook/react)
* [React Router](https://github.com/reactjs/react-router)
* [Express](http://expressjs.com)
* [Feathers](http://feathersjs.com/)
* [Passport](http://passportjs.org) for authentication
* [Babel](http://babeljs.io) for ES6 and ES7 magic
* [Webpack](http://webpack.github.io) for bundling
* [Webpack Dev Middleware](http://webpack.github.io/docs/webpack-dev-middleware.html)
* [Webpack Hot Middleware](https://github.com/glenjamin/webpack-hot-middleware)
* [Redux](https://github.com/reactjs/redux)'s futuristic [Flux](https://facebook.github.io/react/blog/2014/05/06/flux.html) implementation
* [Redux Dev Tools](https://github.com/reactjs/redux-devtools) for next generation DX (developer experience). Watch [Dan Abramov's talk](https://www.youtube.com/watch?v=xsSnOQynTHs).
* [React Router Redux](https://github.com/reactjs/react-router-redux) Redux/React Router bindings.
* [ESLint](http://eslint.org) to maintain a consistent code style
* [redux-form](https://github.com/erikras/redux-form) to manage form state in Redux
* [lru-memoize](https://github.com/erikras/lru-memoize) to speed up form validation
* [multireducer](https://github.com/erikras/multireducer) to combine single reducers into one key-based reducer
* [style-loader](https://github.com/webpack/style-loader), [sass-loader](https://github.com/jtangelder/sass-loader) and [less-loader](https://github.com/webpack/less-loader) to allow import of stylesheets in plain css, sass and less,
* [bootstrap-sass-loader](https://github.com/shakacode/bootstrap-sass-loader) and [font-awesome-webpack](https://github.com/gowravshekar/font-awesome-webpack) to customize Bootstrap and FontAwesome
* [react-helmet](https://github.com/nfl/react-helmet) to manage title and meta tag information on both server and client
* [webpack-isomorphic-tools](https://github.com/halt-hammerzeit/webpack-isomorphic-tools) to allow require() work for statics both on client and server
* [mocha](https://mochajs.org/) to allow writing unit tests for the project.
* [Universal](https://medium.com/@mjackson/universal-javascript-4761051b7ae9) rendering
 
## Installation

```bash
npm install -g yarn && yarn
```
### Database
love Sequelize? => remove `feathers-nedb` dependencies 
- schema/model `User` (eg: `/api/database/User.js`),
- use your favorite adapter in /api/services/

- [feathers-memory](https://github.com/feathersjs/feathers-memory)
- [feathers-mongodb](https://github.com/feathersjs/feathers-mongodb)
- [feathers-mongoose](https://github.com/feathersjs/feathers-mongoose)
- [feathers-nedb](https://github.com/feathersjs/feathers-nedb)
- [feathers-rethinkdb](https://github.com/feathersjs/feathers-rethinkdb)
- [feathers-sequelize](https://github.com/feathersjs/feathers-sequelize)
- [feathers-waterline](https://github.com/feathersjs/feathers-waterline)
- [feathers-localstorage](https://github.com/feathersjs/feathers-localstorage) (A client side service based on feathers-memory that persists to LocalStorage)
- ...

## Running Dev Server

```bash
npm run dev
```
 
### Redux DevTools

[Redux Devtools](https://github.com/gaearon/redux-devtools) are enabled by default in development
[Redux DevTools chrome extension](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd)
see `/webpack/dev.config.js`...

## Production

```bash
npm run build
npm run start
```

or 

```
yarn
````

#### Routing and HTML return

The primary section of `server.js` generates an HTML page with the contents returned by `react-router`.
First we instantiate an `ApiClient`, a facade that both server and client code use to talk to the API server.
On the server side, `ApiClient` is given the request object so that it can pass along the session cookie to the API server to maintain session state.
We pass this API client facade to the `redux` middleware so that the action creators have access to it.

[server-side data fetching](#server-side-data-fetching), wait for the data to be loaded, and render the page with the now-fully-loaded `redux` state.

The last interesting bit of the main routing section of `server.js` is that we swap in the hashed script and css from the `webpack-assets.json` that the Webpack Dev Server – or the Webpack build process on production – has spit out on its last run. You won't have to deal with `webpack-assets.json` manually because [webpack-isomorphic-tools](https://github.com/halt-hammerzeit/webpack-isomorphic-tools) take care of that.

We also spit out the `redux` state into a global `window.__data` variable in the webpage to be loaded by the client-side `redux` code.

#### Server-side Data Fetching

The [redux-connect](https://www.npmjs.com/package/redux-connect) package exposes an API to return promises that need to be fulfilled before a route is rendered. It exposes a `<ReduxAsyncConnect />` container, which wraps our render tree on both [server](https://github.com/erikras/react-redux-universal-hot-example/blob/master/src/server.js) and [client] (client.js). 
More documentation is available on the [redux-connect](https://www.npmjs.com/package/redux-connect) page.

#### Client Side

The client side entry point is reasonably named `client.js`. All it does is load the routes, initiate `react-router`, rehydrate the redux state from the `window.__data` passed in from the server, and render the page over top of the server-rendered DOM. This makes React enable all its event listeners without having to re-render the DOM.

#### Redux Middleware

The middleware, [`clientMiddleware.js`](clientMiddleware.js), serves two functions:

1. To allow the action creators access to the client API facade. Remember this is the same 
  on both the client and the server, and cannot simply be `import`ed because it holds the cookie needed to maintain session on server-to-server requests.
2. To allow some actions to pass a "promise generator", a function that takes the API client 
  and returns a promise. Such actions require three action types, 
    * the `REQUEST` action that initiates the data loading, 
    * a `SUCCESS` and `FAILURE` action that will be fired depending on the result of the promise.
    * another approach were discussed [here](https://github.com/reactjs/redux/issues/99),
3. middleware way feels cleanest

#### Redux Modules

- `src/redux/modules`
- isolate concerns within a Redux application 
- [Ducks Docs](https://github.com/bertho-zero/ducks-modular-redux) and provide feedback.

#### API Server

- server-side
- express, koa, all fits
- connect to database 
- provide authentication and session management 

#### Getting data and actions into components

- [Redux](https://github.com/gaearon/redux) library.
- package the component and its wrapper in the same js file.
- encapsulate component and bound to the `redux` actions and state.

#### Styles

- [local styles](https://medium.com/seek-ui-engineering/the-end-of-global-css-90d2a4a06284)
- [css-loader](https://github.com/webpack/css-loader). The way it works is that you import your stylesheet at the top of the `render()` function in your React Component, and then you use the classnames returned from that import. Like so:

```javascript
render() {
const styles = require('./App.scss');
...
```

Then you set the `className` of your element to match one of the CSS classes in your SCSS file, and you're good to go!

```jsx
<div className={styles.mySection}> ... </div>
```

#### Alternative to Local Styles

If you'd like to use plain inline styles this is possible with a few modifications to your webpack configuration.

**1. Configure Isomorphic Tools to Accept CSS**

In `webpack-isomorphic-tools.js` add **css** to the list of style module extensions

```javascript
    style_modules: {
      extensions: ['less','scss','css'],
```

**2. Add a CSS loader to webpack dev config**

In `dev.config.js` modify **module loaders** to include a test and loader for css

```javascript
  module: {
    loaders: [
      { test: /\.css$/, loader: 'style-loader!css-loader'},
```

**3. Add a CSS loader to the webpack prod config**

You must use the **ExtractTextPlugin** in this loader. In `prod.config.js` modify **module loaders** to include a test and loader for css

```javascript
  module: {
    loaders: [
      { test: /\.css$/, loader: ExtractTextPlugin.extract('style-loader', 'css-loader')},
```

**Now you may simply omit assigning the `required` stylesheet to a variable and keep it at the top of your `render()` function.**

```javascript
render() {
require('./App.css');
require('aModule/dist/style.css');
...
```

**NOTE** In order to use this method with **scss or less** files one more modification must be made. In both `dev.config.js` and `prod.config.js` in the loaders for less and scss files remove 

1. `modules`
2. `localIdentName...`

Before:
```javascript
{ test: /\.less$/, loader: 'style!css?modules&importLoaders=2&sourceMap&localIdentName=[local]___[hash:base64:5]!autoprefixer?browsers=last 2 version!less?outputStyle=expanded&sourceMap' },
```
After:
```javascript
{ test: /\.less$/, loader: 'style!css?importLoaders=2&sourceMap!autoprefixer?browsers=last 2 version!less?outputStyle=expanded&sourceMap' },
```

After this modification to both loaders you will be able to use scss and less files in the same way as css files.

#### Unit tests
```
npm test
```
[Mocha](https://mochajs.org/)
[Karma](http://karma-runner.github.io/0.13/index.html)
[Test Utilities](http://facebook.github.io/react/docs/test-utils.html) from Facebook api like `renderIntoDocument()`.

