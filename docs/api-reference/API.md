# API Reference for API

The API package can be accessed via the `ping` API:

```javascript
import { ping } from 'lambdagrid-mfi';
ping('API', method, arg1, arg2...);
```

## Methods

### `request`

Send API requests. This is often used by AppState readers and writers to sync local application state with a remote database, in addition to event listeners that are meant to trigger API requests as a response to emitted events.

```javascript
ping('API', 'request', name, arg1, arg2, argN...);
```

Where `name` is the name of the request, and `argN` is any additional arguments that the specified request requires.

Returns a Promise with the response body as the resolving callback's first argument.

Example:

```javascript
const name = 'Coco';
const breed = 'Lakeland Terrior';
ping('API', 'request', 'newly born dog', { name, breed })
  .then(dog => ping('AppState', 'write', 'new dog', dog));
```

### `add requests`

Configure API requests that are accessible via `request`.

```javascript
ping('API', 'add requests', {
  name1: requestFunction1,
  name2: requestFunction2,
  nameN: requestFunctionN,
});
```

`nameN` is the identifier for the request that is specified in invocations of `request`.

`requestFunctionN` is a function that takes any number of arguments that you wish, and returns a Promise. The first argument specified in this function's definition will be supplied by the fourth argument in the `request` invocation, as the first, second & third arguments are `'API', 'request', nameOfRequest` respectively. The second argument in this function's definition is the fifth in the `request` invocation, and so forth.

Example:

```javascript
ping('API', 'add requests', {
  'new dog': dog => {
    const url = 'https://api.your-domain.com/dogs';
    return ping('API', 'send http request', 'POST', url, { body: dog });
  }
});
```

### `send http request`

Dispatches an HTTP request with `window.fetch`. Returns a Promise.
