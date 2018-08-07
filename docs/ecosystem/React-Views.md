# React Views

React Views are stateless React components which are consumed by Pagelets. Pagelets use a React View as the rendering component. Pagelets turn App State into data that can be consumed by React Views.

You can make React Views from available Bootstrap-compatible React components, provided free and open source from LambdaGrid, or by making custom React components.

## Free open source Bootstrap-compatible React components

You can import all the Bootstrap components like this:

```javascript
import components from 'lambdagrid-mfi/react-bootstrap-components';
```

### Layout

LambdaGrid doesn't provide special components for layout, but you can use `<div>`s with Bootstrap layout classes. Here's an example:

```javascript
function exampleLayoutComponent(props) {
  return (<div className='container'>
    <div className='row'>
      <div className='col col-lg-2'>
      </div>
      <div className='col-md-auto'>
      </div>
      <div className='col col-lg-2'>
      </div>
    </div>
  </div>);
}
```

### Forms

#### Text input

Ordinary text input ([example](#)):

```javascript
<FieldGroup
  id="formControlsText"
  type="text"
  label="Text"
  placeholder="Enter text"
/>
```

Email text input ([example](#)):

```javascript
<FieldGroup
  id="formControlsEmail"
  type="email"
  label="Email address"
  placeholder="Enter email"
/>
```

Password text input ([example](#)):

```javascript
<FieldGroup
  id="formcontrolspassword"
  label="password"
  type="password"
/>
```

#### File upload

([Example](#)).

```javascript
<FieldGroup
  id="formControlsFile"
  type="file"
  label="File"
  help="Example block-level help text here."
/>
```

#### Checkbox

One checkbox ([example](#)):

```javascript
<Checkbox checked readOnly>
  Example checkbox!
</Checkbox>
```

Several checkboxes ([example](#)):

```javascript
<FormGroup>
  <Checkbox inline>1</Checkbox>
  <Checkbox inline>2</Checkbox>
  <Checkbox inline>3</Checkbox>
</FormGroup>
```

#### Radio

One radio ([example](#)):

```javascript
<Radio checked readOnly>
  Example radio!
</Radio>
```

Several checkboxes ([example](#)):

```javascript
<FormGroup>
  <Radio name="radioGroup" inline>1</Radio>
  <Radio name="radioGroup" inline>2</Radio>
  <Radio name="radioGroup" inline>3</Radio>
</FormGroup>
```

#### Select

Single select ([example](#)):

```javascript
<FormGroup controlId="formControlsSelect">
  <ControlLabel>Select</ControlLabel>
  <FormControl componentClass="select" placeholder="select">
    <option value="select">select</option>
    <option value="other">...</option>
  </FormControl>
</FormGroup>
```

Multiple select ([example](#)):

```javascript
<FormGroup controlId="formControlsSelectMultiple">
  <ControlLabel>Multiple select</ControlLabel>
  <FormControl componentClass="select" multiple>
    <option value="select">select (multiple)</option>
    <option value="other">...</option>
  </FormControl>
</FormGroup>
```

#### Textarea

([Example](#)).

```javascript
<FormGroup controlId="formControlsTextarea">
  <ControlLabel>Textarea</ControlLabel>
  <FormControl componentClass="textarea" placeholder="textarea" />
</FormGroup>
```
