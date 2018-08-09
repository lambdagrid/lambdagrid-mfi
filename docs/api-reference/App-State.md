# API Reference for App State

The App State package can be accessed via the `ping` API:

```javascript
import { ping } from 'lambdagrid-mfi';
ping('AppState', method, arg1, arg2...);
```

## Updaters

Updaters execute all changes to application state, including creates, updates, and deletes. They are synchronous and relate only to local state. Synchronizing with a remote database or remote API is done with Synchronizers, detailed below.

Updaters are primarily consumed by Pagelets.

### Get an Updater

Other packages can talk to AppState and request an application state update by getting an Updater:

```javascript
// in another package other than AppState
ping('AppState', 'get updater', name);
```

The `name` is the name of the updater and is required.

Returns a function that can be invoked to trigger an update, if an updater for the given name is found. Otherwise, throws an error. This function can take any or no arguments, and the arguments themselves are specified by whoever created the Updater.

Example:

```javascript
const changeName = ping('AppState', 'get updater', 'change dog name');
changeName({ id: 123, name: 'Captain Woofers' });
```

### Create Updaters

By default, there are no updaters. You'll have to add updaters if you want other packages to be able to trigger updates to your application state.

```javascript
ping('AppState', 'set updaters', {
  updaterName1: updaterFunction1,
  updaterName2: updaterFunction2,
  updaterName3: updaterFunctionN,
});
```

The `updaterNameN` is the identifing `name` in `ping('AppState', 'get updater', name)`.

The `updaterFunctionN` is a function that takes `previousState` as its first argument, and any additional arguments that might be helpful. For instance, if you want an Updater to remove an item from a list, an additional argument that the Updater could require is an index or an id. The updater function must return the next state.

Example:

```javascript
function changeDogName(prevState, { id, name }) {
  const nextState = prevState.find(dog => dog.get('id') == id).update('name', () => name);
  return nextState;
}

ping('AppState', 'set updaters', { 'change dog name': changeDogName });
```

Note that when you create an Updater, the Updater's first argument is always the previous state. The second argument is the first argument from the invoker, the third argument is the second argument from the invoker, etc.

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
