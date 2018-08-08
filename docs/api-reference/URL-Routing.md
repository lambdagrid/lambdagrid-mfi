# API Reference for URL Routing

The URL Routing package can be accessed via the `ping` API:

```javascript
import { ping } from 'lambdagrid-mfi';
ping('UrlRouting', method, arg1, arg2...);
```

## Setting Routes

You can set routes like this:

```javascript
ping('UrlRouting', 'set routes', {
  route1: reactComponent1,
  route2: reactComponent2,
  routeN: reactComponentN,
});
```

`routeN` is a regular expression in the form of a string. `reactComponentN` is a React component that renders in the main viewport if the regular expression in `routeN` matches the current URL path. This React component could be any React component, whether from React Views, or from Layouts, or even from a third party react component. It could also be a Pagelet.

Example:

```javascript
ping('UrlRouting', 'set routes', {
  '^/login$': ping('Pagelets', 'get pagelet', 'Login'),
  '^/announcements$': ping('ReactViews', 'get view', 'Announcements'),
  '^/dashboard$': ping('Pagelets', 'get pagelet', 'Dashboard'),
});
```

## Setting the 404 page

You can set the 404 page like this:

```javascript
ping('UrlRouting', 'set 404', component);
```

Where `component` is any Pagelet, React View, or any React component.

Example:

```javascript
const pageNotFound = ping('ReactViews', 'get view', 'FailWhail');
ping('UrlRouting', 'set 404', pageNotFound);
```

## Setting automatic 301 redirects

You can set automatic 301 redirects like this:

```javascript
ping('UrlRouting', 'set redirects', {
  regex1: redirect1,
  regex2: redirect2,
  regexN: redirectN,
});
```

Where `regexN` is a regular expression that will be tested against the current URL path, and `redirectN` is a function which takes the current URL path and returns the URL path that you want the URL Router to redirect the end user to.

Example:

```javascript
function removeTrailingSlash(oldPath) {
  const newPath = oldPath.slice(0, oldPath.length-1);
  return newPath;
}

ping('UrlRouting', 'set redirects', {
  '/$': removeTrailingSlash,
});
```
