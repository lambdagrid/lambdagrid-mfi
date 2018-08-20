# Debounce user events

You can debounce high-frequency user events, like `scroll` or `resize` or `keypress`, by using a `debounce` implementation inside of your pagelet's `props` implementation:

```javascript
import _ from 'lodash';

const updateList = () => ping('AppState', 'write', 'update list');

ping('Pagelets', 'init pagelet', {
  transform: () => ({
    onScroll: _.debounce(updateList, 1000),
  }),
});
```
