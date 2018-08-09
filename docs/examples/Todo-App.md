# Todo App

Here's some sample source code for a todo app. Requirements:

* The app should display a list of "todos"
* Each todo has two components: a checkbox and a value
* Users can toggle the checkbox
* Users can click on the value to edit it
* When editing a todo value, users can hit "enter" to submit the change or "escape" to cancel
* Below the todos list is a filter with three links: "show all," "show complete," and "show incomplete"
* When a user clicks a filter link, the todo list updates appropriately

## First, let's create writers in AppState

```javascript
ping('AppState', 'set writers', {
  cancelEditable, // when users hit "escape" while editing a todo value
  onEditableChange, // when users type while editing a todo value
  setFilter, // when users click a filter button
  onEditableSubmit, // when users hit "enter" while editing a todo value
  toggleComplete, // when users toggle a checkbox
  createEditable, // when users click a todo value
});
```

## Next, let's flesh out the AppState Writers

```javascript
function cancelEditable(state, index) {
  return state.setIn(['todos', index, 'isEditing'], false);
}

function onEditableChange(state, index, newEditableValue) {
  return state.setIn(['todos', index, 'editableValue'], newEditableValue);
}

function setFilter(state, nextFilter) {
  return state.set('filter', nextFilter);
}

function onEditableSubmit(state, index) {
  const newValue = state.getIn(['todos', index, 'editableValue']);
  const update = todo => todo.set('isEditing', false).set('value', newValue);
  return state.updateIn(['todos', index], update);
}

function toggleComplete(state, index) {
  return state.updateIn(['todos', index, 'isComplete'], x => !x);
}

function createEditable(state, index) {
  const initialEditableValue = state.getIn(['todos', index, 'value']);
  const update = todo => todo.set('isEditing', true).set('editableValue', initialEditableValue);
  return state.updateIn(['todos', index], update);
}
```

## Next, let's create the React View

```javascript
function TodoApp(props) {
  return (<div>
    <TodoList props={props} />
    <Filter props={props} />
  </div>);
}

ReactViews.registerPageletBody('TodoApp', TodoApp);
```

We need to create two components to make our TodoApp: `TodoList` and `Filter`.

Let's start with `Filter` first:

```javascript
function Filter(props) {
  return (<p>
    <a href="" onClick={() => props.setFilter('all')}>
      Show all
    </a>
    <span> | </span>
    <a href="" onClick={() => props.setFilter('complete')}>
      Show complete
    </a>
    <span> | </span>
    <a href="" onClick={() => props.setFilter('incomplete')}>
      Show incomplete
    </a>
  </p>);
}
```

Now, let's do `TodoList`:

```javascript
function TodoList(props) {
  return (<div>
    <p>Showing {props.filter}</p>
    <ul>
      {props.getFilteredTodos(props.todos, TodoItem)}
    </ul>
  </div>);
}
```

To complete the view, let's do `TodoItem`:

```javascript
function TodoItem(props) {
  const editable = (<li>
    <form onSubmit={props.onEditableSubmit}>
      <FormGroup controlId="editable-todo-item">
        <FormControl
          type="text"
          value={props.editableValue}
          onChange={props.onEditableChange} />
      </FormGroup>
    </form>
  </li>);

  const readOnly = (<li key={props.index}>
    <Checkbox checked={props.isComplete} onChange={props.toggleComplete}>
      <span onClick={props.onReadOnlyClick}>
        {props.value}
      </span>
    </Checkbox>
  </li>);

  return props.isEditing ? editable : readOnly;
}
```

## Finally, let's connect the dogs with a Pagelet

```javascript
const TodoApp = ping('Pagelets', 'create pagelet', {
  isAuthorized: () => true, // anyone is authorized to access this pagelet
  view: ping('ReactViews', 'get view', 'TodoApp'),
  transform,
});

ping('Pagelets', 'set pagelets', { TodoApp });
```

We need to flesh out `transform`:

```javascript
// we'll use `write` a lot, so we'll make life easier and bind it to a function
const write = ping.bind(null, 'AppState', 'write');

function transform(state) {
  return {
    filter: state.get('filter'),
    setFilter: write('setFilter'),
    getFilteredTodos: getFilteredTodos(state),
  };
}
```

We set only three props, `filter` and `setFilter` and `getFilteredTodos`, because those were the only props needed by the ReactView from the top level of `props`.

However, we also need to further specify the value of `getFilteredTodos` because this function will set the props for each individual todo item:

```javascript
function getFilteredTodos(state) {
  return function(TodoItem) {
    const filter = state.get('filter');
    const filterer = filter == 'all' ? () => true
      : filter == 'complete' ? todo => todo.get('isComplete') === true
      : todo => todo.get('isComplete') === false;

    const selectedTodos = todos.filter(filterer);
    const renderedTodos = selectedTodos.map((todo, index) => TodoItem({
      index,
      value: todo.get('value'),
      isComplete: todo.get('isComplete'),
      isEditing: todo.get('isEditing'),
      editableValue: todo.get('editableValue'),
      onEditableSubmit: () => write('onEditableSubmit')(index),
      toggleComplete: () => write('toggleComplete')(index),
      onReadOnlyClick: () => write('createEditable')(index),
      onEditableChange: e => onEditableChange(e, index),
    }));

    return renderedTodos;
  };
}
```

The `TodoItem` has a lot more props needed, so we add a lot more props here.

The last thing we need to do is define `onEditableChange`:

```javascript
function onEditableChange(event, index) {
  const escapeKeyPressed = event.keyCode == 27;
  if (escapeKeyPressed) {
    write('cancelEditable')(index);
  } else {
    write('onEditableChange')(index, event.target.value);
  }
}
```

## Final result

Here's our code for defining our application state:

```javascript
import { ping } from 'lambdagrid-mfi';

// First, let's create the updaters

function cancelEditable(state, index) {
  return state.setIn(['todos', index, 'isEditing'], false);
}

function onEditableChange(state, index, newEditableValue) {
  return state.setIn(['todos', index, 'editableValue'], newEditableValue);
}

function setFilter(state, nextFilter) {
  return state.set('filter', nextFilter);
}

function onEditableSubmit(state, index) {
  const newValue = state.getIn(['todos', index, 'editableValue']);
  const update = todo => todo.set('isEditing', false).set('value', newValue);
  return state.updateIn(['todos', index], update);
}

function toggleComplete(state, index) {
  return state.updateIn(['todos', index, 'isComplete'], x => !x);
}

function createEditable(state, index) {
  const initialEditableValue = state.getIn(['todos', index, 'value']);
  const update = todo => todo.set('isEditing', true).set('editableValue', initialEditableValue);
  return state.updateIn(['todos', index], update);
}

ping('AppState', 'set writers', {
  cancelEditable,
  onEditableChange,
  setFilter,
  onEditableSubmit,
  toggleComplete,
  createEditable,
});

// Then, let's create authenticators.

function anyUser(state) {
  return true;
}

function atLeastReadAccess(state) {
  return state.getIn(['auth', 'access levels', 'read']);
}

function atLeastWriteAccess(state) {
  return state.getIn(['auth', 'access levels', 'write']);
}

function atLeastAdmin(state) {
  return state.getIn(['auth', 'access levels', 'admin']);
}

ping('AppState', 'set authorizers', {
  anyUser,
  atLeastReadAccess,
  atLeastWriteAccess,
  atLeastAdmin,
});
```

Here's our code for defining our React View:

```javascript
import { ReactViews } from 'lambdagrid-mfi';
import React from 'react';
import {
  FormGroup,
  FormControl,
  Checkbox,
} from 'lambdagrid/react-components'

function TodoItem(props) {
  const editable = (<li>
    <form onSubmit={props.onEditableSubmit}>
      <FormGroup controlId="editable-todo-item">
        <FormControl
          type="text"
          value={props.editableValue}
          onChange={props.onEditableChange} />
      </FormGroup>
    </form>
  </li>);

  const readOnly = (<li key={props.index}>
    <Checkbox checked={props.isComplete} onChange={props.toggleComplete}>
      <span onClick={props.onReadOnlyClick}>
        {props.value}
      </span>
    </Checkbox>
  </li>);

  return props.isEditing ? editable : readOnly;
}

function TodoList(props) {
  return (<div>
    <p>Showing {props.filter}</p>
    <ul>
      {props.getFilteredTodos(props.todos, TodoItem)}
    </ul>
  </div>);
}

function Filter(props) {
  return (<p>
    <a href="" onClick={() => props.setFilter('all')}>
      Show all
    </a>
    <span> | </span>
    <a href="" onClick={() => props.setFilter('complete')}>
      Show complete
    </a>
    <span> | </span>
    <a href="" onClick={() => props.setFilter('incomplete')}>
      Show incomplete
    </a>
  </p>);
}

function TodoApp(props) {
  <TodoList props={props} />
  <Filter props={props} />
}

ReactViews.registerPageletBody('TodoApp', TodoApp);
```

And finally, here's our code for defining our Pagelet:

```javascript
import { ping } from 'lambdagrid-mfi';

const write = ping.bind(null, 'AppState', 'write');

function onEditableChange(event, index) {
  const escapeKeyPressed = event.keyCode == 27;
  if (escapeKeyPressed) {
    write('cancelEditable')(index);
  } else {
    write('onEditableChange')(index, event.target.value);
  }
}

function getFilteredTodos(state) {
  return function(TodoItem) {
    const filter = state.get('filter');
    const filterer = filter == 'all' ? () => true
      : filter == 'complete' ? todo => todo.get('isComplete') === true
      : todo => todo.get('isComplete') === false;

    const selectedTodos = todos.filter(filterer);
    const renderedTodos = selectedTodos.map((todo, index) => TodoItem({
      index,
      value: todo.get('value'),
      isComplete: todo.get('isComplete'),
      isEditing: todo.get('isEditing'),
      editableValue: todo.get('editableValue'),
      onEditableSubmit: () => write('onEditableSubmit')(index),
      toggleComplete: () => write('toggleComplete')(index),
      onReadOnlyClick: () => write('createEditable')(index),
      onEditableChange: e => onEditableChange(e, index),
    }));

    return renderedTodos;
  };
}

function transform(state) {
  return {
    filter: state.get('filter'),
    setFilter: write('setFilter'),
    getFilteredTodos: getFilteredTodos(state),
  };
}

const TodoApp = ping('Pagelets', 'create pagelet', {
  isAuthorized: ping('AppState', 'get authorizer', 'anyUser'),
  view: ping('ReactViews', 'get view', 'TodoApp'),
  transform,
});

ping('Pagelets', 'set pagelets', { TodoApp });
```
