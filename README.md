![frontexpress](http://fontmeme.com/embed.php?text=frontexpress&name=Atype%201%20Light.ttf&size=90&style_color=6F6F75)

 Minimalist front end router framework à la [express](http://expressjs.com/)

 [![Build Status](https://travis-ci.org/camelaissani/frontexpress.svg?branch=master)](https://travis-ci.org/camelaissani/frontexpress)
 [![Code Climate](https://codeclimate.com/github/camelaissani/frontexpress/badges/gpa.svg)](https://codeclimate.com/github/camelaissani/frontexpress)
 [![Coverage Status](https://coveralls.io/repos/github/camelaissani/frontexpress/badge.svg?branch=master)](https://coveralls.io/github/camelaissani/frontexpress?branch=master)
 ![Coverage Status](https://david-dm.org/camelaissani/frontexpress.svg)

Let's assume having this html page:

```html
<html>
    <body>
        <button type="button" data-url="http://httpbin.org/get?page=Page1">Page 1</button>
        <button type="button" data-url="http://httpbin.org/get?page=Page2">Page 2</button>
        <button type="button" data-url="http://httpbin.org/get?page=Page3">Page 3</button>
        <button type="button" data-url="http://httpbin.org/status/401">Restricted area</button>

        <div class="content"></div>
    </body>
</html>
```

Here the code managing routes front-end side:

```js
import frontexpress from 'frontexpress';

// Frontend application
const app = frontexpress();

// listen HTTP 401 error
// display an alert on 401 UNAUTHORIZED
app.use((req, res, next) => {
    if (res.status === 401) {
        window.alert('Restricted area!!!');
    } else {
        next();
    }
});

// listen HTTP GET requests from http://httpbin.org/get
// update html page with response content
app.get('http://httpbin.org/get', (req, res, next) => {
    const obj = JSON.parse(res.responseText);
    document.querySelector('.content').innerHTML = `<h1>${obj.args.page}</h1>`;
    next();
});

// start listening frontend application requests
app.listen(() => {
    // Register website buttons
    const buttons = document.querySelectorAll('button');
    for (let i=0; i<buttons.length; i++) {
        const button = buttons[i];
        const url = button.dataset.url;
        button.addEventListener('click', () => {
            // make an ajax call
            app.httpGet(url);
        });
    }
});
```

## Installation

```bash
$ npm install frontexpress
```

## Quick Start

 The quickest way to get started with frontexpress is to clone the [frontexpress-demo](https://github.com/camelaissani/frontexpress-demo) repository.

## Tests

 Clone the git repository:

```bash
$ git clone git@github.com:camelaissani/frontexpress.git
$ cd frontexpress
```

 Install the dependencies and run the test suite:

```bash
$ npm install
$ npm test
```


## Routing samples

### Disclaimer

>
> In this first version of frontexpress, the API is not completely the mirror of the expressjs one.
>
> There are some missing methods. Currently, the use, get, post... methods having a middlewares array as parameter are not available.
> The string pattern to define route paths is not yet implemented.
>
> Obviously, the objective is to have the same API as expressjs when the methods make sense browser side.
>

### Basic routing

Routing allows to link the frontend application with HTTP requests to a particular URI (or path).
The link can be specific to an HTTP request method (GET, POST, and so on).

The following examples illustrate how to define simple routes.

Listen an HTTP GET request on URI (/):

```js
app.get('/', (req, res) => {
  window.alert('Hello World');
});
```

Listen an HTTP POST request on URI (/):

```js
app.post('/', (req, res) => {
  window.alert('Got a POST request at /');
});
```

### Route paths

Route paths, in combination with a request method, define the endpoints at which requests can be made.
Route paths can be strings (see basic routing section), or regular expressions.

This route path matches all GET request paths which start with (/api/):

```js
app.get(/^api\//, (req, res) => {
  console.log(`api was requested ${req.uri}`);
});
```

### Route handlers

You can provide multiple callback functions to handle a request. Invoking ```next()``` function allows to pass the control to subsequent routes.
Whether ```next()``` method is not called the handler chain is stoped

```js
const h1 = (req, res, next) => { console.log('h1!'); next(); };
const h2 = (req, res, next) => { console.log('h2!') };
const h3 = (req, res, next) => { console.log('h3!'); next(); };

app.get('/example/a', h1);
app.get('/example/a', h2);
app.get('/example/a', h3);
```

A response to a GET request on path (/example/a) displays:

```
h1!
h2!
```

h3 is ignored because ```next()``` function was not invoked.

### app.route()

You can create chainable route handlers for a route path by using ```app.route()```.

```js
app.route('/book')
 .get((req, res) => { console.log('Get a random book') })
 .post((req, res) => { console.log('Add a book') })
 .put((req, res) => { console.log('Update the book') });
```

### frontexpress.Router

Use the ```frontexpress.Router``` class to create modular, mountable route handlers.

Create a router file named ```birds.js``` in the app directory, with the following content:

```js
import frontexpress from 'frontexpress';

const router = frontexpress.Router();

// middleware that is specific to this router
router.use((req, res, next) => {
  console.log(`Time: ${Date.now()}`);
  next();
});

// react on home page route
router.get('/', (req, res) => {
  document.querySelector('.content').innerHTML = '<p>Birds home page</p>';
});

// react on about route
router.get('/about', (req, res) => {
  document.querySelector('.content').innerHTML = '<p>About birds</p>';
});

export default router;
```

Then, load the router module in the app:

```js
import birds  from './birds';
...
app.use('/birds', birds);
```

The app will now be able to react on requests (/birds) and (/birds/about)

## API

|               | Method        | Short description |
| ------------- | --------------| ----------------- |
|Frontexpress |||
||[frontexpress()](https://github.com/camelaissani/frontexpress/blob/master/docs/frontexpress.md#frontexpress-1)|Creates an instance of application|
||[frontexpress.Router()](https://github.com/camelaissani/frontexpress/blob/master/docs/frontexpress.md#frontexpressrouter)|Creates a Router object|
||[frontexpress.Middleware()](https://github.com/camelaissani/frontexpress/blob/master/docs/frontexpress.md#frontexpressmiddleware)|Creates a Middleware object|
||||
|Application |||
||[set(setting, value)](https://github.com/camelaissani/frontexpress/blob/master/docs/application.md#applicationsetsetting-val)|Assigns a setting|
||[listen(callback)](https://github.com/camelaissani/frontexpress/blob/master/docs/application.md#applicationlistencallback)|Starts the application|
||[route(uri)](https://github.com/camelaissani/frontexpress/blob/master/docs/application.md#applicationrouteuri)|Gets a Router initialized with a root path|
||[use(uri, middleware)](https://github.com/camelaissani/frontexpress/blob/master/docs/application.md#applicationuseuri-middleware)|Sets a middleware|
||||
||[get(uri, middleware)](https://github.com/camelaissani/frontexpress/blob/master/docs/application.md#applicationgeturi-middleware-applicationposturi-middleware)|Applies a middleware on given path for a GET request|
||[post(uri, middleware)](https://github.com/camelaissani/frontexpress/blob/master/docs/application.md#applicationgeturi-middleware-applicationposturi-middleware)|Applies a middleware on given path for a POST request|
||[put(uri, middleware)](https://github.com/camelaissani/frontexpress/blob/master/docs/application.md#applicationgeturi-middleware-applicationposturi-middleware)|Applies a middleware on given path for a PUT request|
||[delete(uri, middleware)](https://github.com/camelaissani/frontexpress/blob/master/docs/application.md#applicationgeturi-middleware-applicationposturi-middleware)|Applies a middleware on given path for a DELETE request|
||||
||[httpGet(request, success, failure)](https://github.com/camelaissani/frontexpress/blob/master/docs/application.md#applicationhttpgetrequest-success-failure-applicationhttppostrequest-success-failure)|Invokes a GET ajax request|
||[httpPost(request, success, failure)](https://github.com/camelaissani/frontexpress/blob/master/docs/application.md#applicationhttpgetrequest-success-failure-applicationhttppostrequest-success-failure)|Invokes a POST ajax request|
||[httpPut(request, success, failure)](https://github.com/camelaissani/frontexpress/blob/master/docs/application.md#applicationhttpgetrequest-success-failure-applicationhttppostrequest-success-failure)|Invokes a PUT ajax request|
||[httpDelete(request, success, failure)](https://github.com/camelaissani/frontexpress/blob/master/docs/application.md#applicationhttpgetrequest-success-failure-applicationhttppostrequest-success-failure)|Invokes a DELETE ajax request|
||||
|Router |||
||[use(middleware)](https://github.com/camelaissani/frontexpress/blob/master/docs/router.md#routerusemiddleware)|Sets a middleware|
||[all(middleware)](https://github.com/camelaissani/frontexpress/blob/master/docs/router.md#routerallmiddleware)|Sets a middleware on all HTTP method requests|
||||
||[get(uri, middleware)](https://github.com/camelaissani/frontexpress/blob/master/docs/router.md#routergeturi-middleware-routerposturi-middleware)|Applies a middleware on given path for a GET request|
||[post(uri, middleware)](https://github.com/camelaissani/frontexpress/blob/master/docs/router.md#routergeturi-middleware-routerposturi-middleware)|Applies a middleware on given path for a POST request|
||[put(uri, middleware)](https://github.com/camelaissani/frontexpress/blob/master/docs/router.md#routergeturi-middleware-routerposturi-middleware)|Applies a middleware on given path for a PUT request|
||[delete(uri, middleware)](https://github.com/camelaissani/frontexpress/blob/master/docs/router.md#routergeturi-middleware-routerposturi-middleware)|Applies a middleware on given path for a DELETE request|
||||
|Middleware |||
||[entered(request)](https://github.com/camelaissani/frontexpress/blob/master/docs/middleware.md#middlewareenteredrequest)|Invoked by the app before an ajax request is sent|
||[exited(request)](https://github.com/camelaissani/frontexpress/blob/master/docs/middleware.md#middlewareexitedrequest)|Invoked by the app before a new ajax request is sent|
||[updated(request, response)](https://github.com/camelaissani/frontexpress/blob/master/docs/middleware.md#middlewareupdatedrequest-response)|Invoked by the app after an ajax request has responded|
||[failed(request, response)](https://github.com/camelaissani/frontexpress/blob/master/docs/middleware.md#middlewarefailedrequest-response)|Invoked by the app after an ajax request has failed|
||[next()](https://github.com/camelaissani/frontexpress/blob/master/docs/middleware.md#middlewarenext)|Allows to break the middleware chain execution|


### middleware function

After registering a middleware function, the application invokes it with these parameters:

```js
  (request, response, next) => {
    next();
  }
```

**request**: `Object`, the ajax request information sent by the app

**response**: `Object`, the response of request

**next**: `Function`, the `next()` function to call to not break the middleware execution chain


### request object

```js
  {
    method,
    uri,
    headers,
    data,
    history: {
      state,
      title,
      uri
    }
  }
```

**method**: `String`, HTTP methods 'GET', 'POST'...

**uri**: `String`, path

**headers**: `Object`, custom HTTP headers

**data**: `Object`, data attached to the request

**history**: `Object`, object with properties state, title and uri

**If the history object is set, it will activate the browser history management.** See [browser pushState() method](https://developer.mozilla.org/en-US/docs/Web/API/History_API#The_pushState()_method) for more information about state, title, and uri (url).

> uri and history.uri can be different.


### response object

```js
  {
    status,
    statusText,
    responseText,
    errorThrown,
    errors
  }
```

**status**: `Number`, HTTP status 200, 404, 401, 500...

**statusText**: `String`

**responseText**: `String` response content

**errorThrown**: `Object` exception thrown (if request fails)

**errors**: `String` error description (if request fails)


## License

 [MIT](LICENSE)
