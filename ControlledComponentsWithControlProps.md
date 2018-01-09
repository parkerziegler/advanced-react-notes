## Make Controlled React Components with Control Props

[**Controlled components**](https://reactjs.org/docs/forms.html#controlled-components) are a key concept in React. When using controlled components, you are responsible for managing the `state` of the component manually. We can use this pattern to do something like preventing a user form clicking a button too many times. For example:

```javascript
class Toggle extends React.Component {
  static defaultProps = {
    defaultOn: false,
    onToggle: () => {},
    onReset: () => {}
  };

  initialState = { on: this.props.defaultOn };
  state = this.initialState;

  /* for the two methods below, check if they are managed by a control prop on. if so,
  we'll use the supplied control prop with callbacks. if not, we can use internal
  setState with the state value of on */
  reset = () => {
    if (this.isOnControlled()) {
      this.props.onReset(!this.props.on);
    } else {
      this.setState(this.initialState, () => this.props.onReset(this.state.on));
    }
  };

  toggle = () => {
    if (this.isOnControlled()) {
      this.props.onToggle(!this.props.on);
    } else {
      this.setState(
        ({ on }) => ({ on: !on }),
        () => this.props.onToggle(this.state.on)
      );
    }
  };

  getTogglerProps = ({ onClick, ...props } = {}) => ({
    onClick: compose(onClick, this.toggle),
    "aria-expanded": this.state.on,
    ...props
  });

  /* a utility method to check if state is controlled internally or
  managed by a control prop */
  isOnControlled() {
    return this.props.on !== undefined;
  }

  render() {
    return this.props.render({
      on: this.isOnControlled() ? this.props.on : this.state.on,
      toggle: this.toggle,
      reset: this.reset,
      getTogglerProps: this.getTogglerProps
    });
  }
}

class App extends React.Component {
  initialState = { timesClicked: 0, on: false };
  state = this.initialState;

  /* handle toggle will increment the count onClick.
    past 4 clicks we'll prevent toggling. */
  handleToggle = () => {
    this.setState(({ timesClicked, on }) => ({
      timesClicked: timesClicked + 1,
      on: timesClicked >= 4 ? false : !on
    }));
  };

  handleReset = () => this.setState(initialState);

  render() {
    const { timesClicked, on } = this.state;

    return (
      <Toggle
        on={on}
        onToggle={this.handleToggle}
        onReset={this.handleReset}
        render={toggle => (
          <div>
            <Switch
              {...toggle.getTogglerProps({
                on: toggle.on
              })}
            />
            {timesClicked > 4 ? (
              <div>
                Whoa, you've clicked too much!
                <br />
                <button onClick={toggle.reset}>reset</button>
              </div>
            ) : timesClicked > 0 ? (
              <div>Click count: {timesClicked}</div>
            ) : null}
          </div>
        )}
      />
    );
  }
}
```

The key thing to realize here is that we have flexibility over how the `state` of `<Toggle />` gets controlled. If we pass `<Toggle />` a control `prop` of `on`, then we are controlling the `state` of `<Toggle />` according to that `prop`. However, this implementation doesn't require the user to not specify an `on` `prop`. In that case, `on` gets managed behind the scenes by the `<Toggle />` component itself. The key is to run a check before calling `this.setState` to see if `on` is controlled by a control `prop` or controlled internally. To do this, we use the utility `this.isOnControlled`, which just checks if `this.props.on` is `undefined`.
