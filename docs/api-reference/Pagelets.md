# API Reference for Pagelets

The Pagelets package can be accessed via the `ping` API:

```javascript
import { ping } from 'lambdagrid-mfi';
ping('Pagelets', method, arg1, arg2...);
```

## Methods

### `get pagelet`

You can get a Pagelet like this:

```javascript
ping('Pagelets', 'get pagelet', name);
```

Where `name` is the name of the Pagelet that you want. This returns the Pagelet if a Pagelet with your provided name is found, otherwise it returns an error.

### `init pagelet`

You can initialize a pagelet like this:

```javascript
const newPagelet = ping('Pagelets', 'init pagelet', {
  isAuthorized: authorizationFunction,
  view: reactView,
  props: propsFn,
});
```

`authorizationFunction` is an optional predicate which takes current application state and returns a boolean to decide whether the user is authorized to see the view.

`reactView` is a required view from ReactViews, or it could be any React component.

`props` is a required function that takes the current application state and returns the props that will be sent to the `reactView`. This function re-runs every time app state updates and the `reactView` is mounted in the DOM.

Example:

```javascript
const DogsList = ping('Pagelets', 'create pagelet', {
  isAuthorized: ping('AppState', 'get authorizer', 'is logged in'),
  view: ping('AppState', 'get view', 'DogsList'),
  props: () => ({
    dogs: ping('AppState', 'read', 'list of dogs'),
  }),
});
```

### `create pagelets`

You can create pagelets like this:

```javascript
ping('Pagelets', 'create pagelets', {
  name: newPagelet,
});
```

Where `name` is the identifier that other packages can use to find this pagelet, and `newPagelet` is an initialized pagelet created with the `init pagelet` method.

**What's the difference between `init pagelet` and `create pagelets`?**

They have different functions. `init pagelet` will return a React component that's integrated with `react-redux`. `create pagelets` will register the pagelets so that other packages can discover it with `get pagelet`.

Example:

```javascript
ping('Pagelets', 'create pagelets', { DogsList });
```
