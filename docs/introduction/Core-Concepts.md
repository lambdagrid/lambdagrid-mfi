# Core Concepts

## Stateless functions as business logic

Most business logic in LambdaGrid can be expressed as a stateless JavaScript function.

For instance, expressing how application state should update after a user adds a new item to a list, often called a reducer:

```javascript
function addItem(prevState, newItem) {
  const nextState = prevState.update('items', items => items.push(newItem));
  return nextState;
}
```

Another simple example is a React component:

```javascript
function Item(props) {
  return <div>{props.text}</div>;
}
```

The specific types of functions you'll write will depend on which package will receive the functions, since each package owns a different part of your UI. Your packages responsible for your views will require functions that return React components, while your packages responsible for updating your application state will require reducer functions.

## Packages

### A package for each domain

A LambdaGrid UI is the composition of many packages, each of which owning a specific domain. For instance, there is a package for organizing app state, another for reading from and writing to local storage, and another for React views.

### Composing packages

Packages are most useful when combined with other packages. A UI will combine packages together to create end-to-end functionality.

Packages can be added, replaced, or removed as you see fit. The plug-and-play nature of packages is intended to allow your UI to scale up alongside your requirements. We want the composition of packages to prevent developers from having to fight against a rigid framework.

If you're missing a package for an important feature, or if you'd like a new way to compose your packages, file an issue on the LambdaGrid GitHub repo and we'll help out.

### Using packages

Each package has its own API, which reflects how each package has its own purpose. Usually, the APIs are for adding stateless JavaScript functions to the package. For instance, the app state package exposes an API for adding functions that update the app's global state:

```javascript
import { AppState } from 'lambdagrid-mfi';

AppState.registerUpdaters(
  'add a dog': (state, newDog) => state.update('dogs', dogs => dogs.push(newDog)),
  'remove a dog': (state, name) => state.filter('dogs', dogs => dogs.filter(d => d.name != name))
);
```

### Managing your package code

The way we recommend you manage your code, your business logic expressed as small amounts of JavaScript, is in a mono-repo.

```
└── package-configs
    ├── app-state
    │   └── index.js
    ├── react-views
    │   └── index.js
    └── pagelets
        └── index.js
```

In this example, there are three packages being configured: `app-state`, `react-views`, and `pagelets`. There are three directories of the same names, all sitting inside a parent directory `package-configs`.

Within each directory is an `index.js` file. This is the only required file for each package config. You can add other files too and import them into `index.js`. The only limitation is that no files should request a file in a separate package's directory, which will break your UI.
