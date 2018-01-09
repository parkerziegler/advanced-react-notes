## Use Prop Getters with Render Props

**Prop Getters** is a technique that allows you to hide away the implementation details of how specific prop collections get passed to components, while still affording some composability to your `props`. In particular, they are useful when you have prop collections that need to interact with `props` passed by the parent. For example:

```javascript
class Toggle extends React.Component {
  /* a method for getting togglerProps.
    by default, props is an empty object */
  getTogglerProps = ({ onClick, ...props } = {}) => {
    return {
      "aria-expanded": this.state.on,
      onClick: (...args) => {
        /* check if an onClick prop was passed by the parent.
                if so, execute it with passed arguments and then resume
                standard execution by calling this.toggle */
        onClick && onClick(...args);
        this.toggle(...args);
      },
      // spread all other props from the parent
      ...props
    };
  };

  render() {
    return this.props.render({
      on: this.state.on,
      toggle: this.toggle,
      // use a prop getter to supply props
      getTogglerProps: this.getTogglerProps
    });
  }
}

// rather than passing toggleProps, let's pass a function to act as a prop getter
function App() {
  return (
    <Toggle
      onToggle={on => console.log("toggle", on)}
      render={({ on, toggle, togglerProps }) => (
        <div>
          <Switch on={on} {...getTogglerProps()} />
          <hr />
          {/* add some functionality to onClick by specifying it as an arg to getToggleProps */}
          <button
            {...getTogglerProps({
              onClick: () => alert("hi")
            })}
          >
            {on ? "on" : "off"}
          </button>
        </div>
      )}
    />
  );
}
```

You can also abstract this into a utility to make it easier to manage. For example:

```javascript
const compose = (...fns) => (...args) => fns.forEach(fn => fn && fn(...args));

// this.getTogglerProps becomes...
getTogglerProps = ({ onClick, ...props } = {}) => {
  return {
    "aria-expanded": this.state.on,
    onClick: compose(onClick, this.toggle),
    // spread all other props from the parent
    ...props
  };
};
```
