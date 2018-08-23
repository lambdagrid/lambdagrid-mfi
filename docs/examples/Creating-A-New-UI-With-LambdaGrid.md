# Creating a new UI with LambdaGrid

Let's create a new UI together using LambdaGrid!

## Hello World

We'll begin by creating an end-to-end Hello World app to demonstrate the skeleton of LambdaGrid.

First, create a new repository for your LambdaGrid UI, and create a `src/index.js` file. All of your configurations go into this file.

> *What about directory structure?*
>
> You can organize your files however you want, as long as they're inside the `src` folder and the `src/index.js` file will import it. We'll give you some good directory structure guidelines in this tutorial so you don't have to worry about accidentally mis-organizing your files.

Inside `src/index.js`, copy and paste the following code:

```javascript
import { ping, finish } from 'lambdagrid-mfi';
import React from 'react';

ping('ReactViews', 'create views', {
  HelloWorld: () => <div>hello world</div>,
});

ping('UrlRouting', 'create routes', {
  '^/$': ping('ReactViews', 'get view', 'HelloWorld'),
});

finish();
```

> We used the `create views` method for the ReactViews package, for which you can see the API reference [here](https://docs.lambdagrid.com/api-reference/reactviews#create-views). We also used the `create routes` method for UrlRouting, for which you can see the API reference [here](https://docs.lambdagrid.com/api-reference/urlrouting#create-routes).

Almost done with our Hello World app. Finally, create a `src/index.html` file and paste the following:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-beta.2/css/bootstrap.min.css" >
    <title>explore demo</title>
</head>
<body>
  <div id="root"></div>
</body>
</html>
```

Then run `npm start` in the root directory of your repository.

Check your browser, and you should now see "hello world" rendering inside of your new LambdaGrid application at `localhost:8080`!

### Tinkering with Hello World

Before moving on to the next level of complexity, we encourage you tinker with your existing app. Try some of these different options:

1. Change the React code inside the `HelloWorld` React component. You can add links, change the text, etc.
2. Try removing the `finish()` invocation at the end of `src/index.js`. Your UI should now not be running at all!

## Directory structure

If your UI will be non-trivial, we recommend against putting all your UI configurations inside `src/index.js` and instead splitting them into multiple files.

Although technically a directory structure can be anything as long as `src/index.js` requires the files and the files themselves live in `src`, the canonical way to organize your files is like this:

```
src
├── API
│   └── index.js
├── AppState
│   └── index.js
├── Pagelets
│   └── index.js
├── ReactViews
│   └── index.js
├── index.html
└── index.js
```

Both `index.js` and `index.html` are one level inside of `src`. `index.js` ends up requiring the rest of the files like this:

```javascript
// src/index.js

import { finish } from 'lambdagrid-mfi';
import './ReactViews';
import './API';
import './AppState';
import './Pagelets';

finish();
```

In each of the subdirectories, the files create the resources for their respective packages. For instance, the AppState directory's `index.js` creates the readers and writers, the API directory's `index.js` creates the requests, the ReactViews directory's `index.js` creates the React views, and the Pagelets directory's `index.js` creates the pagelets.

Within these separate directories, you could further split your code however you'd like. For instance, you could create your AppState writers in a separate file and import it in `src/AppState/index.js`.
