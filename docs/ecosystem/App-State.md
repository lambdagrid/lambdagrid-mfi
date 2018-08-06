# App State

App State is the database of your application. It handles reading data from and writing data to your application state.

Assuming your MFI config names your App State `AppState`, you can start configuring your application state in your app state's `index.js` file with:

```javascript
import { AppState } from 'lambdagrid';
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
AppState.registerUpdaters({
  'add a dog': (state, newDog) => state.update('dogs', dogs => dogs.push(newDog)),
  'remove a dog': (state, name) => state.filter('dogs', dogs => dogs.filter(d => d.name != name))
});
```

Note that the updaters as functions take two arguments: the previous state and new data sent to the updater. Updaters return the new state.

Updaters are similar to reducers in Redux, but they're called Updaters because they also include the Redux action component in addition to the reducer component.

## Invoking an action to modify state

In other packages, you will want to invoke an updater to trigger state changes. You can do so as follows:

```javascript
// inside another package's index.js
const addDog = AppState.getUpdater('add a dog');
addDog({ name: 'Fluffy', size: 'medium' });
```

Our aim is to allow other packages to invoke updates to App State while not needing to know the implementation details in App State.
