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

Example:

```javascript
const name = 'Coco';
const breed = 'Lakeland Terrior';
ping('API', 'request', 'newly born dog', { name, breed });
```
