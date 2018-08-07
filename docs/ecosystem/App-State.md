# App State

App State is the database of your application. It handles reading data from and writing data to your application state.

You can start configuring your application state in your app state's `index.js` file with:

```javascript
import { AppState } from 'lambdagrid-mfi';
```

App State converts your app state into an [Immutable.js object](https://facebook.github.io/immutable-js/docs/) to expose more powerful APIs to manipulate state while also reaping the benefits of immutability.

## Loading initial state

```javascript
// inside App State's index.js
AppState.createInitialState({
  dogs: [
    {
      name: 'Bowser',
      size: 'small'
    },
    {
      name: 'Dexter',
      size: 'large'
    }
  ]
});
```

## Updaters

Updaters are stateless functions which update the app state when new information comes in. Create them like this:

```javascript
// inside App State's index.js

import 'Immutable' from 'immutable';

function addDogUpdater(prevState, newDogToAdd) {
  const newDog = Immutable.fromJS(newDogToAdd);
  const nextState = prevState.update('dogs', dogs => dogs.push(newDog));
  return nextState;
}

function removeDogUpdater(prevState, name) {
  const notPickedByUser = dog => dog.get('name') != name;
  const nextState = prevState.update('dogs', dogs => dogs.filter(notPickedByUser));
  return nextState;
}

AppState.registerUpdaters({
  'add a dog': addDogUpdater,
  'remove a dog': removeDogUpdater,
});
```

Note that the updaters as functions take at least one argument, the previous state. You can add extra arguments that will be passed by the updater invoker, if you want to pass custom data to the updater. In this case, the updater's invoker for `add a dog` will pass configs for a new dog.

Updaters are similar to reducers in Redux, but they're called Updaters because they also include the Redux action component in addition to the reducer component.

## Invoking an action to modify state

In other packages, you will want to invoke an updater to trigger state changes. You can do so as follows:

```javascript
// inside another package's index.js
const addDog = AppState.getUpdater('add a dog');
addDog({ name: 'Fluffy', size: 'medium' });
```

In this example, `{ name: 'Fluffy', size: 'medium' }` is the config that gets sent to the `add a dog` updater as the second argument. If the `addDog` invocation included a second argument, that argument would appear as the third argument in `add a dog`.

Our aim is to allow other packages to invoke updates to App State while not needing to know the implementation details in App State.

## Authorizers

Authorizers are stateless functions which return a boolean, also called "predicates." Authorizers take app state as an argument, and then return `true` or `false` depending on whether or not the app state is sufficient to authorize access to whatever package invokes the authorizer.

For instance, let's say we have the following authorizers:

```javascript
function anyUser(state) {
  return true;
}

function atLeastReadAccess(state) {
  return state.getIn(['auth', 'access levels', 'read']);
}

function atLeastWriteAccess(state) {
  return state.getIn(['auth', 'access levels', 'write']);
}

function atLeastAdmin(state) {
  return state.getIn(['auth', 'access levels', 'admin']);
}
```

Pagelets can select one of these authorizers to pick an access level for users requesting the pagelets. The login screen, for instance, would probably use `anyUser`. `atLeastReadAccess` could be great for viewing a list of records. `atLeastWriteAccess` could be great for modifying the list of records. `atLeastAdmin` could be used for credit cards.
