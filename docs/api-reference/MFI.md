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

## Configuration

This is an example configuration:

```json
{
  "version": "0.0.1",
  "packages": {
    "AppState": {
      "packageTemplate": "URL for the AppState package's base template",
      "userConfigs": "URL for the simple JS which configures the package"
    },
    "ReactViews": {
      "packageTemplate": "URL for the AppState package's base template",
      "userConfigs": "URL for the simple JS which configures the package"
    },
    "Pagelets": {
      "packageTemplate": "URL for the AppState package's base template",
      "userConfigs": "URL for the simple JS which configures the package"
    }
  }
}
```

The `version` is the version of MFI to run.

The `packages` is a mapping of package names, as they would be defined in the rest of your package config files, to package configs.

The `packages[packageName].packageTemplate` is a URL representing the location of the package's template. A package template is all the package's infrastructure and boilerplate. Package templates often are, but are not required to be, coupled with a `packages[packageName].userConfigs` URL representing the location of the package's user configurations expressed as small amounts of JavaScript.
