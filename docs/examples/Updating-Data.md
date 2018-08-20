# Updating data

We want users to be able to update their data, otherwise our UI is just a read-only dashboard. This is handled by the `Pagelets` package passing in event handlers via `props` to the React views.

Let's say we have the following React view:

```javascript
ping('ReactViews', 'create views', {
  login: props => (<Form onSubmit={props.onSubmit}>
    <FormGroup>
      <Label>Email</Label>
      <Input
        type="email"
        id="email"
        value={props.email}
        onChange={e => props.onChange('email', e.target.value)}
        placeholder="name@company.com"
      />
    </FormGroup>
    <FormGroup>
      <Label>Password</Label>
      <Input
        type="password"
        id="password"
        value={props.password}
        onChange={e => props.onChange('password', e.target.value)}
      />
    </FormGroup>
    <Button type="submit">Login</Button>
  </Form>),
});
```

We want to be able to submit the inputted login credentials.

## Simplest: Add dummy event handlers to props

We can add dummy event handlers to the props generator for the pagelet:

```javascript
ping('Pagelets', 'init pagelet', {
  view: ping('ReactViews', 'get view', 'login');
  transform: () => ({
    email: 'dummy email',
    password: 'dummy password',
    onChange: (field, value) => console.log('onChange:', field, value),
    onSubmit: () => console.log('onSubmit'),
  }),
});
```

When you trigger either the `onChange` or `onSubmit` handlers, the console will log your messages.

## Harder: Use AppState `readers` and `writers`

We can update our pagelet to use AppState `readers` to read from application state, and `writers` to write to application state.

```javascript
ping('Pagelets', 'init pagelet', {
  view: ping('ReactViews', 'get view', 'login');
  transform: () => ({
    email: ping('AppState', 'read', 'login form', 'email'),
    password: ping('AppState', 'read', 'login form', 'password'),
    onChange: (field, value) => ping('AppState', 'write', 'login form', field, value),
    onSubmit: () => ping('AppState', 'write', 'login'),
  }),
});

ping('AppState', 'create readers', {
  'login form': (state, field) => state.getIn(['forms', 'login', field]),
});

ping('AppState', 'create writers', {
  'login form': (state, field, value) => state.setIn(['forms', 'login', field], value),
  'login': state => state.update('user', user => user
    .set( 'email', state.getIn(['forms', 'login', 'email']) )
    .set( 'password', state.getIn(['forms', 'login', 'password']) )
  ),
});
```

## Hardest: Sync with remote database via API request

Most of us need to sync our UI's application state with an API server so our databases can be updated too.

First, we'll extend the writer to dispatch an API request to log in the user. Second, we'll configure the API request. Third and finally, we'll handle the response body from the API.

```javascript
// First, extend the writer to dispatch a request

function login(state) {
  const email = state.getIn(['forms', 'login', 'email']);
  const password = state.getIn(['forms', 'login', 'password']);
  return ping('API', 'request', login, { email, password })
    .then(() => ping('AppState', 'write', 'user info', { email, password }));
}

ping('AppState', 'create writers', {
  login,
});

// Second, configure an API request

function loginRequest({ email, password }) {
  const url = 'https://api.your-domain.com/auth';
  return ping('API', 'send http request', 'POST', url, { email, password });
}

ping('API', 'create requests', {
  login: loginRequest,
});

// Third, handle the response

ping('AppState', 'create writers', {
  'user info': state => state.update('user', user => user
    .set( 'email', state.getIn(['forms', 'login', 'email']) )
    .set( 'password', state.getIn(['forms', 'login', 'password']) )
});
```
