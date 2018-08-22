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

When you run `npm start` in the root directory of your repository, you should be able to see your read-only React view on `localhost:8080`.

> We used the `get view` method with the ReactViews package. Learn more about it [here](https://docs.lambdagrid.com/api-reference/reactviews#get-view).

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
    name: ping('AppState', 'read', 'name');
  }),
});
```

We're telling the AppState package to read something for us, with the reader identifier `name`.

We won't be able to see our changes live on `localhost:8080` until after step 1.6, so keep going!

### Step 1.5: Provide the data from AppState

We can configure AppState to provide this data. If the shape of your application state is complicated, we recommend using an Immutable.JS object to represent your state. In this simplified example, we'll use a plain JavaScript object.

To provide the data from AppState, we need to create a reader, which is a gateway for other packages to request to read application state.

```javascript
ping('AppState', 'create readers', {
  name: state => state.name;
});
```

Note that the implementation details of your reader will depend on the shape of your application state.

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

## Step 2: Read-only feature extended for edge cases

## Step 3: Read-only feature with data fetched from an API

## Step 4: Writing data without API requests

## Step 5: Writing data with API requests
