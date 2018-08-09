# API Reference for App State

The App State package can be accessed via the `ping` API:

```javascript
import { ping } from 'lambdagrid-mfi';
ping('AppState', method, arg1, arg2...);
```

## Reading state from AppState with Readers

You can read from your application state with Readers. This also includes fetching data from an API server to populate your application state.

### Consuming Readers

Pagelets are the main consumers of Readers. You can consume a Reader like this:

```javascript
ping('AppState', 'read', nameOfReadRequest, arg1, arg2, argN...);
```

Where `nameOfReadRequest` is a required string which is a query name that is configured in AppState, and `argN` is an optional additional parameter.

### Creating Readers

You create Readers in AppState like this:

```javascript
ping('AppState', 'set readers', {
  name1: readerFunction1,
  name2: readerFunction2,
  nameN: readerFunctionN,
});
```

Where `nameN` is the `name` of a query that consumers specify with `ping('AppState', 'read', name)`, and `readerFunctionN` is a function that takes the current app state as the first argument and returns either the updated app state, or a promise which eventually resolves to the updated app state. Additional arguments in the definition of `readerFunctionN` are optional but can be useful to make state queries more dynamic.

Example:

```javascript
function getDogList(state) {
  if (state.has('dogs')) {
    return state.get('dogs');
  } else {
    return ping('API', 'get server', 'dogs server', 'get all dogs')
      .then(dogs => state.update('dogs', () => Immutable.fromJS(dogs)));
  }
}

function getDogDetail(state, id) {
  const dogWithId = dog => dog.get('id') === id;
  const dogs = state.get('dogs');
  const dog = dogs.find(dogWithId);

  if (dog.has('owner history') && dog.has('medical history')) {
    return dog;
  } else {
    return ping('API', 'get server', 'dogs server', 'get dog', id)
      .then(newDog => state.updateIn(['dogs', dogs.findIndex(dogWithId)], newDog));
  }
}

ping('AppState', 'set readers', {
  getDogList,
  getDogDetail,
});
```

## Storage

You can specify a storage option to persist data between tabs and sessions. This helps particularly for keeping users logged in while between sessions.

Currently, the only option we support is `window.localStorage`, but we'll be adding other options soon.

### Picking your storage option

By default, App State uses `localStorage`. If you want to be explicit, you can set it like this:

```javascript
ping('AppState', 'set storage', 'localStorage');
```

### Picking data to persist between sessions

You can specify paths in your application state to persist to your storage selection like this:

```javascript
ping('AppState', 'set persisted paths', [
  path1,
  path2,
  pathN,
]);
```

Where `pathN` is an array of strings or numbers, which indicate the path to traverse your application state to select parts of your state to persist.

Example:

```javascript
ping('AppState', 'set persisted paths', [
  ['user', 'auth'],
  ['tabs', 'currently selected'],
  ['timezone'],
]);
```

## Initial state

You can specify an initial state for each session:

```javascript
ping('AppState', 'set initial state', initialState);
```

Where `initialState` is an Immutable.JS object.

Note that `initialState` won't replace any state that's populated from your storage option! So if all your initial state is already rehydrated from, say, `localStorage`, then `initialState` will do nothing.

Example:

```javascript
ping('AppState', 'set initial state', Immutable.fromJS({
  preferences: {
    mainTabIndex: 0,
    defaultNewDogName: "Captain Woofers",
  },
}));
```

## Authorizers

Authorizers are predicates that take in app state as its sole argument, and then return a boolean to dictate whether or not the app state is sufficient to authorize a user. They're primarily consumed by Pagelets.

### Get an Authorizer

You can get an Authorizer like this:

```javascript
ping('AppState', 'get authorizer', name);
```

Where `name` is the name of the Authorizer. Returns the Authorizer function if there's a created Authorizer with that name, otherwise returns an error.

Example:

```javascript
const fullyPublic = ping('AppState', 'get authorizer', 'anyone');
const loginRequired = ping('AppState', 'get authorizer', 'is logged in');
```

### Create Authorizers

You can create Authorizers like this:

```javascript
ping('AppState', 'set authorizers', {
  authorizerName1: authorizerFunction1,
  authorizerName2: authorizerFunction2,
  authorizerNameN: authorizerFunctionN,
});
```

The `authorizerNameN` is the identifying `name` in `ping('AppState', 'get authorizer', name)`.

The `authorizerFunctionN` is a function that takes `currentState` as its only argument. The updater function must return a boolean.

Example:

```javascript
function letEveryoneIn() {
  return true;
}

function loginRequired(state) {
  return state.getIn(['user', 'is logged in']) === true;
}

ping('AppState', 'set authorizers', {
  'anyone': letEveryoneIn,
  'is logged in': loginRequired,
});
```

## Synchronizers

Synchronizers help to sync your UI's application state with the data in your database behind an API server. There are two types of Synchronizers:

1. Readers, which take data from the API server and load it into App State
2. Writers, which take user requests to change, update, or delete data, and propagate those changes back to the API server

### Readers

Readers are consumed by Pagelets, but configured in AppState.

#### Consumption

Pagelets ask for data from AppState like this:

```javascript
ping('AppState', 'read', pathToData);
```

Where `pathToData` is an array of strings and numbers that represents the path to traverse AppState to get your desired data. This returns an array with two items. The first is the data, if data exists at your specified path, otherwise it is `null`. The second is a boolean, either `true` if a Reader Synchronizer is fetching your data, or `false` if not.

Example:

```javascript
const path = ['dogs', 123, 'history'];
const [data, fetching] = ping('AppState', 'read', path);
if (fetching) {
  return DogHistoryView({ fetching });
} else {
  return DogHistoryView(data);
}
```

In this example, if `fetching` is `true`, then an API request is being dispatched in the background. When the API request returns, it updates AppState, which will trigger the Pagelet to re-run its reader. This time, the application state has all the data the Pagelet wants, so `data` is populated and `fetching` is false.

#### Configuration

For the consumption of Readers to work, Readers need to be configured like this:

```javascript
ping('AppState', 'set readers', {
  name1: readerFunction1,
  name2: readerFunction2,
  nameN: readerFunctionN,
});

ping('AppState', 'set readers', [
  [pathFunction1, readerFunction1],
  [pathFunction2, readerFunction2],
  [pathFunctionN, readerFunctionN],
]);
```

Here, `pathFunctionN` is a function which takes one argument, a wildcard placeholder, and returns a path of strings, numbers, and the wildcard placeholder. You can think of it as a regular expression, except for matching a path array instead of a string. The `readerFunctionN` is a function that runs when the `pathFunctionN` function matches a path array sent into `ping('AppState', 'read', pathArray)`. It takes the current application state as its only argument, and it returns a Promise for an response from the API package, plus serializing and adding the data to the correct spot in the app state.

Example:

```javascript
function dogDetail(wildcard) {
  return ['dogs', wildcard];
}

function dogList(wildcard) {
  return ['dogs'];
}

function getDogList(state) {
  return ping('API', 'get service', 'dog service')
    .then(dogs => state.update('dogs', Immutable.fromJS(dogs)));
}

ping('AppState', 'set readers', [
  [dogDetail, getDogList],
  [dogList, getDogList],
]);
```

### Writers

Writers are also consumed by Pagelets but configured in AppState.

#### Consumption

Pagelets
