# API Reference for MFI

## Ping other packages

MFI exposes one primary function: `ping`.

```javascript
import { ping } from 'lambdagrid-mfi';
ping(packageName, methodName, arg1, arg2, argN...);
```

`ping` is the way for packages to talk to one another.

`packageName` is the name of the package and is required.

`methodName` is the name of the method and is required.

`argN` is an additional argument. The required nature of these additional arguments depend on the `methodName` that is being pinged.

Examples:

```javascript
const reactViewExample = ping('ReactViews', 'get view', 'ExampleView');

const updateName = ping('AppState', 'get updater', 'update name');
```
