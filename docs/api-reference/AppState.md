# API Reference for AppState

The AppState package can be accessed via the `ping` API:

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

Example:

```javascript
const id = 123;
ping('AppState', 'read', 'getDogDetail', id);
```

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

## Writing state to AppState with Writers

Along with Readers, you can write state to both your application state and to API servers with Writers.

### Consuming Writers

Pagelets are the main consumers of Writers. You can consume a Writer like this:

```javascript
ping('AppState', 'write', nameOfWriteRequest, arg1, arg2, argN...);
```

Where `nameOfWriteRequest` is a required string which is a query name that is configured in AppState, and `argN` is an optional additional parameter.

Email:

```javascript
const id = 123;
ping('AppState', 'write', 'update dog name', id, 'Captain Woofers');
```

### Creating Writers

You can create Writers like this:

```javascript
ping('AppState', 'set writers', {
  name1: writerFunction1,
  name2: writerFunction2,
  nameN: writerFunctionN,
});
```

Where `nameN` is the `name` of a query that consumers specify with `ping('AppState', 'write', name)`, and `writerFunctionN` is a function that takes the current app state as the first argument and returns either the updated app state, or a promise which eventually resolves to the updated app state. Additional arguments in the definition of `writerFunctionN` are optional but can be useful to make state queries more dynamic.

Example:

```javascript
function updateDogName(state, id, name) {
  const dogs = state.get('dogs');
  const dogWithId = d => d.get('id') === id;
  return ping('API', 'get server', 'dogs server', 'update dog', id, { name })
    .then(newDog => state.updateIn(['dogs', dogs.findIndex(dogWithId)], newDog));
}

ping('AppState', 'set writers', {
  'update dog name': updateDogName,
});
```

## Storage

You can specify a storage option to persist data between tabs and sessions. This helps particularly for keeping users logged in while between sessions.

Currently, the only option we support is `window.localStorage`, but we'll be adding other options soon.

### Picking your storage option

By default, AppState uses `localStorage`. If you want to be explicit, you can set it like this:

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
