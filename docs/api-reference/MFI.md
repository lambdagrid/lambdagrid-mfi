# API Reference for MFI

## Methods

### `ping`

MFI's primary function is `ping`. It's the primary way for packages to talk to one another.

```javascript
import { ping } from 'lambdagrid-mfi';
ping(packageName, methodName, arg1, arg2, argN...);
```

`packageName` is the name of the package and is required.

`methodName` is the name of the method and is required.

`argN` is an additional argument. The required nature of these additional arguments depend on the `methodName` that is being pinged.

Examples:

```javascript
const reactViewExample = ping('ReactViews', 'get view', 'ExampleView');

const updateName = ping('AppState', 'get updater', 'update name');
```

### `emit`

MFI's method for publishing events to the event bus, which is powered by [Eev](https://github.com/chrisdavies/eev).

Example:

```javascript
import { emit } from 'lambdagrid-mfi';
emit('dogs-arrived', ['Momo', 'Dexter', 'Clementine']);
```

### `on`

MFI's method for subscribing to events on the event bus, which is powered by [Eev](https://github.com/chrisdavies/eev).

Example:

```javascript
import { on } from 'lambdagrid-mfi';
on('dogs-arrived', dogs => dogs.forEach(d => pet(d)));
```
