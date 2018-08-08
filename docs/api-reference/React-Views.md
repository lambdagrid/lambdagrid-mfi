# API Reference for React Views

The React Views package can be accessed via the `ping` API:

```javascript
import { ping } from 'lambdagrid-mfi';
ping('ReactViews', method, arg1, arg2...);
```

## Getting and setting React views

### Get views

You can get a view like this:

```javascript
ping('ReactViews', 'get view', name);
```

The `name` is the name of the React view that you want to retrieve. This will return a function that takes props and returns a React component, if the `name` corresponds to a registered React view, otherwise it returns an error.

Example:

```javascript
const NameTag = ping('ReactViews', 'get view', 'NameTag');
```

### Set views

You can register a view like this:

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
