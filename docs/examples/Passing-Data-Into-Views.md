# Passing data into views

LambdaGrid hooks up your stateless React views with dynamic data via the `Pagelets` package. If a React view is a "dumb" component that is responsible only for rendering markup and handling user events, not for processing data, then a pagelet is a "smart" component which handles business logic to process data to be consumed by the React view.

Let's say we have the following React view:

```javascript
ping('ReactViews', 'create views', {
  NameTag: props => <div>hi, my name is {props.name}</div>,
});
```

And this React view is hooked directly into your URL routing:

```javascript
ping('UrlRouting', 'create routes', {
  '^/$': ping('ReactViews', 'get view', 'NameTag'),
});
```

## Simplest: hardcoding data to pass into a React view

The simplest way to pass in a `props.name` to the React view is to hardcode it inside the pagelet. First, we'll create a new pagelet, since pagelets are responsible for business logic, and wrap the pagelet around the React view. Next, we'll point the URL router to the pagelet instead of the React view.

```javascript
// First, create the new pagelet

const NameTag = ping('Pagelets', 'init pagelet', {
  view: ping('ReactViews', 'get view', 'NameTag'),
  props: () => ({
    name: 'Alyssa',
  }),
});

ping('Pagelets', 'create pagelets', { NameTag });

// Next, update the URL router

ping('UrlRouting', 'create routes', {
  '^/$': ping('Pagelets', 'get pagelet', 'NameTag'),
});
```

This will hardcode `props.name` to be `Alyssa`. Try changing the pagelet to return a `props.name` to be `Bailey` instead.

## Harder: dynamic data passing into a React view

Most real-world UIs won't hardcode their production data. Let's read it from `AppState`.

First, we'll have to create an application state `reader` to read the `name` attribute from app state. Then, we'll configure the pagelet to consume this reader.

```javascript
// First, create the reader to extract data from AppState

ping('AppState', 'create readers', {
  'name': state => state.get('name'),
});

// Then, have the pagelet use this reader

const NameTag = ping('Pagelets', 'init pagelet', {
  view: ping('ReactViews', 'get view', 'NameTag'),
  props: () => ({
    name: () => ping('AppState', 'read', 'name'),
  }),
});
```

Now, any time the `name` changes in `AppState`, the `NameTag` React view will render the new name.

## Hardest: getting dynamic data via API request into a React view
