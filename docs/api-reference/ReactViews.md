# API Reference for ReactViews package

The ReactViews package can be accessed via the `ping` API:

```javascript
import { ping } from 'lambdagrid-mfi';
ping('ReactViews', method, arg1, arg2...);
```

## Methods

### `get view`

You can get a view like this:

```javascript
ping('ReactViews', 'get view', name);
```

The `name` is the name of the React view that you want to retrieve. This will return a React component, if the `name` corresponds to a registered React view, otherwise it returns an error.

Example:

```javascript
const NameTag = ping('ReactViews', 'get view', 'NameTag');
```

Common use cases:
* From the Pagelets package, to specify a pagelet's view
* From the UrlRouting package, to specify a route's view

### `create views`

You can register views like this:

```javascript
ping('ReactViews', 'set views', {
  viewName1: reactComponent1,
  viewName2: reactComponent2,
  viewName3: reactComponent3,
});
```

The `viewNameN` is the identifying `name` in `ping('ReactViews', 'get view', name)`.

The `reactComponentN` is a function that takes React props as its only argument and returns a React component.

Example:

```javascript
function NameTag(props) {
  return <div>Hi, my name is {props.name}</div>;
}

ping('ReactViews', 'set views', { NameTag });
```
