# koa-load-routes
[![NPM version][npm-image]][npm-url] [![Build Status][travis-image]][travis-url] [![Dependency Status][daviddm-image]][daviddm-url] [![Coverage Status](https://coveralls.io/repos/github/gbahamondezc/koa-load-routes/badge.svg?branch=master)](https://coveralls.io/github/gbahamondezc/koa-load-routes?branch=master)

[Koa](https://github.com/koajs/koa) middleware to load routes from files and directories using [koa-router](https://github.com/alexmingoia/koa-router) module


### Warning
- **Node.js 8.0** and **koa@2.x** or latest are required to use this module.
- **Since v2.0** koa-load-routes not support **Generator Functions** anymore (in favor of [Async Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)).
- **Since v3.0** ***path*** option must be a absolute path
- **Since v3.0** ***firstMiddleware*** option was removed in favor of ***middlewares*** option.

## Installation

```sh
$ npm install --save koa-load-routes
```
or

```sh
$ yarn add koa-load-routes
```

##### Options

- **String ->** *path (required)*
    - Path from where the module will try to load the routes, if is a file,  it will read the routes defined just inside of the file, otherwise, if is a directory, will read all .js files in the directory and load the routes inside each one.

- **Boolean ->** *recursive (optional, default: false)*
  - If this argument is true the module will search route files recursive trhoug directories.

- **Array[Function|AsyncFunction] ->** *middlewares (optional)*
  - A list of middlewares which will be added to each route loaded, can be Array of async functions or a normal functions or mixed, this middlewares will be attached previous to each route, useful by example for authentication middlewares.

- **String ->** *base (optional)*
  - Adds the "base" string at the start of each  loaded route url.

- **Array[any] ->** *args (optional)*
  - Arguments to be pased inside the route main function, each array elements are passed as a independent argument (see examples below).

- **String ->** *suffix (optional, default: '.js')*
  - If this argument is passed the module will load only the files which ends with the passed **suffix + .js**, by default if **suffix** is not supplied will try to load all files with **.js** extension.


### Usage

###### app.js
```js
const Koa = require('koa');
const loader = require('koa-load-routes');
const authMiddleware = require('./auth-middleware');
const path = require('path');

// Some module containing business logic and db access
const Interactors = require('./interactors');

var app = new Koa();

// Loading public routes
app.use(route({
  path: `${path.resolve()}/src/resources/public`,
  recursive: true,
  // Add /api/ at start of each loaded endpoint
  base: 'api',
  // Load files only ending with route.js
  suffix: 'route',
  // Injecting your business logic module
  args: [Interactors]
}));

// Loading private routes with auth middleware
app.use(route({
  path: `${path.resolve()}/src/resources/account`,
  recursive: true,
  base: 'api',
  suffix: 'route',
  middlewares : [authMiddleware()],
  args: [Interactors]
}));

app.listen(3000, () => {
  console.log('server started at http://127.0.0.1:3000/');
});

```

###### src/resources/public/login.route.js

```js
module.exports = function (Interactors) {

  // Authentication endpoint
  this.post('/login', async ctx => {

    let userAccount = await Interactors.userAccount
      .getByEmail(ctx.request.body.email);

    if (!userAccount) {
      ctx.throw(404, 'Account not found');
    }

    if (ctx.body.password !== userAccount.password) {
      ctx.throw(403, 'Wrong Email/Password combination');
    }

   /*
    * You can create some kind of auth token here to
    * send it in body with user account info.
    * This is just an example
    */
    ctx.body = userAccount;
  });



  // Other possible routes examples

  this.get('/hello', (ctx, next) => {
    ctx.body = 'Hello world';
    return next();
  });

  // Routes chain
  this.get('/hello2', (ctx, next) => {
    ctx.body = 'hello world 2';
  })
  .get('/hello3', (ctx, next) => {
    ctx.body = 'hello world 3';
  });

  // Multiple middlewares
  this.post('/hello4', (ctx, next) => {
    console.log('im the first one');
    return next();
  },
  (ctx, next) => {
    ctx.body = 'yey!';
  });
};
```


## License

MIT © [Gonzalo Bahamondez](https://github.com/gbahamondezc/)


[npm-image]: https://badge.fury.io/js/koa-load-routes.svg
[npm-url]: https://npmjs.org/package/koa-load-routes
[travis-image]: https://travis-ci.org/gbahamondezc/koa-load-routes.svg?branch=master
[travis-url]: https://travis-ci.org/gbahamondezc/koa-load-routes
[daviddm-image]: https://david-dm.org/gbahamondezc/koa-load-routes.svg?theme=shields.io
[daviddm-url]: https://david-dm.org/gbahamondezc/koa-load-routes
