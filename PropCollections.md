## Use Prop Collections with Render Props

When using `props` that may be common to several components or HTML elements in the `render` method of your component, it can make sense to group these into **prop collections**. For example:

```javascript
class Toggle extends React.Component {
  // pass common props used by <Switch> and <button> as togglerProps colelction
  render() {
    return this.props.render({
      on: this.state.on,
      toggle: this.toggle,
      // here's our collection of props
      togglerProps: {
        "aria-expanded": this.state.on,
        onClick: this.toggle
      }
    });
  }
}

function App() {
  return (
    <Toggle
      onToggle={on => console.log("toggle", on)}
      render={({ on, toggle, togglerProps }) => (
        <div>
          {/* spread the togglerProps for the elements / components that use it */}
          <Switch on={on} {...togglerProps} />
          <hr />
          <button {...togglerProps}>{on ? "on" : "off"}</button>
        </div>
      )}
    />
  );
}
```

This pattern is particularly useful for grouping collections of `props` that you may want to apply broadly across types of elements, like `aria` attributes for `button`s, `input`s, etc.
