# MFI

MFI stands for Micro Frontend Infrastructure. It's the glue that ties together all the LambdaGrid UI packages.

## Configuration

This is an example configuration:

```json
{
  "version": "0.0.1",
  "packages": {
    "AppState": {
      "packageTemplate": "URL for the AppState package's base template",
      "userConfigs": "URL for the simple JS which configures the package",
      "step": 0
    },
    "ReactViews": {
      "packageTemplate": "URL for the AppState package's base template",
      "userConfigs": "URL for the simple JS which configures the package",
      "step": 0
    },
    "Pagelets": {
      "packageTemplate": "URL for the AppState package's base template",
      "userConfigs": "URL for the simple JS which configures the package",
      "step": 1
    }
  }
}
```

The `version` is the version of MFI to run.

The `packages` is a mapping of package names, as they would be defined in the rest of your package config files, to package configs.

The `packages[packageName].packageTemplate` is a URL representing the location of the package's template. A package template is all the package's infrastructure and boilerplate. Package templates often are, but are not required to be, coupled with a `packages[packageName].userConfigs` URL representing the location of the package's user configurations expressed as small amounts of JavaScript.

The `packages[packageName].step` is a number indicating the position in the runtime initialization process that the package template and user configs run in. For instance, a package with step `0` will be initialized before a package with step `1`, which in turn will be initialized before a third package with step `4`. This way, if the third package relies on the first or second package, MFI can ensure that the first and second packages are ready before the third package initializes. This is useful to ensure upstream packages are prepared to be consumed by downstream packages.
