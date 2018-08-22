# Building Your First Feature with LambdaGrid

To build a feature with LambdaGrid, build the following in this order:

1. A read-only version of your feature visualizing the main use case
2. Extended versions of your feature covering all your edge cases
3. Dynamic data that can be read from an API request
4. User interactions to edit or write data
5. Dispatch API requests to

Note that steps 1-3 are all increasing in complexity for a read-only version of the feature. We'll add write functionality with increasing complexity in steps 4 and 5.

## Step 0: Initial setup

If this is a new UI, check out the [Hello World](https://docs.lambdagrid.com/examples/creating-a-new-ui-with-lambdagrid) to start in the right place!

## Step 1: Read-only feature for main use case

Take a wireframe or draw a single picture of your feature to represent the main use case that you're building. We'll build this first.

The following libraries are available with LambdaGrid, along with their documentation:

* [reactstrap](https://reactstrap.github.io/) for Bootstrap-compatible React components
* [react-content-loader](http://danilowoz.com/create-content-loader/) for placeholder loading
* [Immutable.js](https://facebook.github.io/immutable-js/) for Immutable data APIs
* And more to come!

### Step 1.1: Create the React view

You'll probably use reactstrap most to create your views. Proceed to the next step after you're done marking up your main use case. You should hard-code things like form values and data in this step.

You might have a nametag like this:

```javascript
ping('ReactViews', 'create views', {
  NameTag: () => <div>hi, my name is alyssa</div>,
});
```

Don't forget to update the UrlRouting package to render this view:

```javascript
ping('UrlRouting', 'create routes', {
  '^/$': ping('ReactViews', 'get view', 'NameTag'),
});
```

When you run `npm start` in the root directory of your repository, you should be able to see your read-only React view on `localhost:8080`.

> See the API reference for ReactViews's `get view`  [here](https://docs.lambdagrid.com/api-reference/reactviews#get-view) and for UrlRouting's `create routes` [here](https://docs.lambdagrid.com/api-reference/urlrouting#create-routes).

### Step 1.2: Change the React view to use `props` instead of hardcoding

The next step is to remove the hard-coding from the React view. Replace them all with `props`.

```javascript
ping('ReactViews', 'create views', {
  // Previous:
  // NameTag: () => <div>hi, my name is alyssa</div>,

  // Next:
  NameTag: props => <div>hi, my name is {props.name}</div>,
});
```

We won't be able to see the change live on `localhost:8080` again until we finish the next step, so keep going!

> See the API reference for ReactViews's `create views` [here](https://docs.lambdagrid.com/api-reference/reactviews#create-views).

### Step 1.3: Provide the data through a pagelet

You can supply data and logic to React views by wrapping them in pagelets. Create a pagelet like this:

```javascript
const NameTag = ping('Pagelets', 'init pagelet', {
  view: ping('ReactViews', 'get view', 'NameTag'),
  props: () => ({
    name: 'still alyssa',
  }),
});

ping('Pagelets', 'create pagelets', { NameTag });
```

Now we just need to update the UrlRouting configuration to point to the pagelet instead of the React view:

```javascript
ping('UrlRouting', 'create routes', {
  // Previous:
  // '^/$': ping('ReactViews', 'get view', 'NameTag'),

  // Next:
  '^/$': ping('Pagelets', 'get pagelet', 'NameTag'),
});
```

We removed the hardcoding from the React view and put it into the pagelet. Check out your React view, decoupled from the data, at work on `localhost:8080`!

> See the API references for `init pagelet` [here](https://docs.lambdagrid.com/api-reference/pagelets#init-pagelet) and for `create pagelet` [here](https://docs.lambdagrid.com/api-reference/pagelets#create-pagelets).

### Step 1.4: Request data from AppState

Now let's remove the hardcoding from the pagelet too and move it further upstream, to AppState.

Change the pagelet's `props` function like so:

```javascript
const NameTag = ping('Pagelets', 'init pagelet', {
  view: ping('ReactViews', 'get view', 'NameTag'),
  props: () => ({
    // Previous:
    // name: 'still alyssa',

    // Next:
    name: ping('AppState', 'read', 'name'),
  }),
});
```

We're telling the AppState package to read something for us, with the reader identifier `name`.

> See API reference for AppState's `read` [here](https://docs.lambdagrid.com/api-reference/appstate#read).

We won't be able to see our changes live on `localhost:8080` until after step 1.6, so keep going!

### Step 1.5: Provide the data from AppState

We can configure AppState to provide this data. If the shape of your application state is complicated, we recommend using an Immutable.JS object to represent your state. In the future, we'll support plain JavaScript objects too.

To provide the data from AppState, we need to create a reader, which is a gateway for other packages to request to read application state.

```javascript
ping('AppState', 'create readers', {
  name: state => state.get('name'),
});
```

Note that the implementation details of your reader will depend on the shape of your application state.

> See API reference for `create readers` [here](https://docs.lambdagrid.com/api-reference/appstate#create-readers).

Note also that the key of the reader is `name`, to match the reader identifier used in the pagelet's `props` function.

Just one more step until we can see our changes live on `localhost:8080`!

### Step 1.6: Initialize the data in AppState

The data still needs to originate somewhere, and for now we'll do it directly inside AppState.

```javascript
ping('AppState', 'set initial state', {
  name: "definitely alyssa",
});
```

At this stage, you should finally be able to see your changes showing up on `localhost:8080`!

> See API reference for `set initial state` [here](https://docs.lambdagrid.com/api-reference/appstate#set-initial-state).

## Step 2: Read-only feature extended for edge cases

We want our feature to handle all the different states, or all the different edge cases, that our user could experience.

We recommend first listing all the edge cases, then for each edge case, repeat the steps above.

So, for our NameTag example, let's say we might want to add their age, if we know their age. We'd start with hardcoding the age in the React view, then making the React view dynamic by hardcoding the age in the pagelet, then making the pagelet dynamic by hardcoding the age in AppState's initial state.

## Step 3: Read-only feature with data fetched from an API

We're still hardcoding data in AppState's initial state. Let's remove that hardcoding too and fetch our initial state from an API server.

### Step 3.1: Extend readers to know when they need data

It's the job of the readers in AppState to know whether or not to get data from an API. You can program a reader to trigger an API request if the requested data isn't present locally:

```javascript
ping('AppState', 'create readers', {
  // Previous:
  // name: state => state.get('name'),

  // Next:
  name: state => {
    const name = state.get('name');

    if (!name) {
      ping('API', 'request', 'name')
        .then(response => ping('AppState', 'write', 'name', response.name));
    }

    const defaultName = '';
    return name || defaultName;
  }
});
```

In this example, the reader dispatches an API request if the name isn't present locally. After the request finishes, it then triggers a writer to update the name. Note that we haven't yet written the code for this writer!

> See API references for the API package's `request` [here](https://docs.lambdagrid.com/api-reference/api#request).

### Step 3.2: Configure the API request

We need to create the API request that is called in the reader.

```javascript
ping('API', 'add requests', {
  name: () => {
    const url = 'http://localhost:3000/name';
    return ping('API', 'send http request', 'GET', url);
  },
});
```

You can create the requests however you like, as long as you follow the convention of returning a Promise. One way to do so is to use API's `send http request` method.

> See API reference for API's `add requests` [here](https://docs.lambdagrid.com/api-reference/api#add-requests) and `send http request` [here](https://docs.lambdagrid.com/api-reference/api#send-http-request).

### Step 3.3: Configure the writer

We need to take the data from the API server and add it to our own local application state by using an AppState writer.

Recall this example a few steps previously:

```javascript
ping('AppState', 'create readers', {
  name: state => {
    const name = state.get('name');

    if (!name) {
      ping('API', 'request', 'name')
        // Let's look at this line in particular:
        .then(response => ping('AppState', 'write', 'name', response.name));
    }

    const defaultName = '';
    return name || defaultName;
  }
});
```

We need to actually configure this writer if we want to use it.

```javascript
ping('AppState', 'create writers', {
  name: (state, newName) => state.set('name', newName),
});
```

> See API reference for `create writers` [here](https://docs.lambdagrid.com/api-reference/appstate#create-writers) and for `writers` [here](https://docs.lambdagrid.com/api-reference/appstate#write).

## Step 4: Writing data without API requests

## Step 5: Writing data with API requests
