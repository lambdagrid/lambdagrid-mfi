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
