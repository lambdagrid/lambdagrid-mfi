# API Reference for AppState

The AppState package can be accessed via the `ping` API:

```javascript
import { ping } from 'lambdagrid-mfi';
ping('AppState', method, arg1, arg2...);
```

## Methods

### `read`

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

### `create readers`

You create Readers in AppState like this:

```javascript
ping('AppState', 'create readers', {
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

ping('AppState', 'create readers', {
  getDogList,
  getDogDetail,
});
```

### `write`

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

### `create writers`

You can create Writers like this:

```javascript
ping('AppState', 'create writers', {
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

ping('AppState', 'create writers', {
  'update dog name': updateDogName,
});
```

### `set initial state`

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

### `get authorizer`

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

### `create authorizers`

You can create Authorizers like this:

```javascript
ping('AppState', 'create authorizers', {
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

ping('AppState', 'create authorizers', {
  'anyone': letEveryoneIn,
  'is logged in': loginRequired,
});
```

### `is fetching`

Returns a boolean to indicate whether an API request is underway.

```javascript
ping('AppState', 'is fetching', anyOrAll, request1, request2, requestN...);
```

`anyOrAll` is an enum with value either "any" or "all".

`requestN` is a name of a request that's in progress. These are the names specified by the `name` in `ping('API', 'request', name)`.

If `anyOrAll` is `"any"`, then this returns `true` if at least one of `requestN` is underway, otherwise `false`. Else, if `anyOrAll` is `"all"`, then this returns `true` if all of `requestN` is underway, otherwise `false`.

Example:

```javascript
const fetchingFirstItem = ping(
  'AppState',
  'is fetching',
  'any',
  'eating',
  'drinking water'
);

const fetchingAllItems = ping(
  'AppState',
  'is fetching',
  'all',
  'eating',
  'drinking water'
);
```
