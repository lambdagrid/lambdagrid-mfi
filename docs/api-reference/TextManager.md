# API Reference for TextManager

The TextManager package can be accessed via the `ping` API:

```javascript
import { ping } from 'lambdagrid-mfi';
ping('TextManager', method, arg1, arg2...);
```

## Choose text

You can get text from the Text Manager like this:

```javascript
const text = ping('TextManager', 'get text', textIdentifier);
```

Where `textIdentifier` is the key for the text. The return value is the string that is associated with the `textIdentifier` key.

## Adding, changing, and deleting text

You can add, change, and delete text through the LambdaGrid dashboard. We'll be exposing an API soon as well.
