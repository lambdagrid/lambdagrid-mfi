# Pagelets

Pagelets are the hub of all application logic: turning business data into a format which is useful for consumption by React Views for the user. They're called "Pagelets" because they can be thought of as mini-pages.

Pagelets create a layer of abstraction between App State and React Views so that the two can be further decoupled.

```javascript
import { Pagelets } from 'lambdagrid';
```

While a pagelet consumes both App State and React Views, Pagelets itself is consumed by the URL routing service. When the URL routing service requests a pagelet, the pagelet does the following:

1. Validates the request. If the user is not authenticated, invalidate the request
2. Request the data needed from App State to populate the React Views
3. Create event handlers to respond to user events
4. Return the React View with appropriate data to render in the user's browser

## Creating a pagelet

```javascript
const pageletConfigs = { /* insert configs here */ };
const newPagelet = Pagelets.createPagelet(pageletConfigs);
Pagelets.registerPagelets('PageletName', newPagelet);
```

The `PageletName` is for other packages which consume Pagelets, mainly the URL routing service. The consuming packages use the `PageletName` to find the correct pagelet.

## Pagelet Configs

Here's an example configuration:

```javascript
const pageletConfigs = {
  authorized: function validateTheUserIsAuthenticated() {},
  view: 'SampleReactView',
  transform: function transformAppStateIntoConsumableDataForReactView() {},
};
```

### `pageletConfigs.authenticator`

The `authenticator` config specifies an authentication function to use to verify that the user is allowed to look at the page. We recommend this function come from App State to keep business logic contained within App State and remain agnostic to the implementation details, but you could write your own function within your pagelet if you're not afraid of the coupling.

This function takes the new application state as an input, and returns `true` if the user is authorized and `false` otherwise. It's the downstream consuming service's responsibility to handle the result, whether `true` or `false`.

### `pageletConfigs.view`

The `view` config specifies the name of the registered React View to render, if the `authorized` function returns `true`.

### `pageletConfigs.transform`

The `transform` function specifies a function which takes the new application state, and it returns a `props` object which the specified React View consumes.

The `props` includes two different types of props:

1. Data, like numbers, strings, objects, and arrays
2. Event handlers, to respond to user events like a click, a keypress, or a form submit

It'll be the React View's responsibility to map all the props to the appropriate React elements to render the data correctly and map event handlers correctly.
