# API Reference for Pagelets

The Pagelets package can be accessed via the `ping` API:

```javascript
import { ping } from 'lambdagrid-mfi';
ping('Pagelets', method, arg1, arg2...);
```

## Getting and setting Pagelets

### Getting a Pagelet

You can get a Pagelet like this:

```javascript
ping('Pagelets', 'get pagelet', name);
```

Where `name` is the name of the Pagelet that you want. This returns the Pagelet if a Pagelet with your provided name is found, otherwise it returns an error.

### Setting Pagelets

You can create Pagelets like this:

```javascript
const newPagelet = ping('Pagelets', 'create pagelet', {
  isAuthorized: authorizationFunction,
  view: reactView,
  readers: [
    reader1,
    reader2,
    readerN,
  ],
  transform: transformFunction,
});

ping('Pagelets', 'set pagelets', {
  name: newPagelet,
});
```

`authorizationFunction` is a predicate which takes current application state and returns a boolean to decide whether the user is authorized to see the view.

`reactView` is a view from ReactViews, or it could be any React component.

`readerN` is an AppState reader which specifies the Pagelet's required information, and allows AppState the opportunity to fetch any data the Pagelet needs from an API server.

`transformFunction` is a function that takes the current application state and returns the props that will be sent to the `reactView`. This function re-runs every time app state updates and the `reactView` is mounted in the DOM.

`name` is the identifier that other services can use to find this Pagelet.

Example:

```javascript
const DogsList = ping('Pagelets', 'create pagelet', {
  isAuthorized: ping('AppState', 'get authorizer', 'is logged in'),
  view: ping('AppState', 'get view', 'DogsList'),
  readers: [
    ping('AppState', 'read', 'dogs list')
  ],
  transform: state => ({
    dogs: state.get('dogs').map(dog => ({
      name: dog.get('name'),
      size: dog.get('size'),
    })),
    total: state.get('dogs').count(),
  });
});

ping('Pagelets', 'set pagelets', { DogsList });
```
