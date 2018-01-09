## Use Component State Initializers

Initializing component `state` dynamically via `props` can give your application a lot of flexibility with regards to how it initially renders. Most importantly, it allows your component to hold internal `state` while still giving the user of your component the ability to specify a default `state`. For example:

```javascript
class Toggle extends React.Component {

    // provide defaultProps for all non-required props
    static defaultProps = {
        defaultOn: false
        onToggle: () => {}
        onReset: () => {}
    }

    // initialize state using a prop
    initialState = { on: this.props.defaultOn };
    state = this.initialState;

    getTogglerProps = ({ onClick, ...props } = {}) => {
        return {
            "aria-expanded": this.state.on,
            onClick: (...args) => {
                onClick && onClick(...args);
                this.toggle(...args);
            },
            ...props
        };
    }

    // a new reset method for resetting the toggle to defaultOn
    reset = () => this.setState(initialState, () => this.props.onReset(this.state.on))

    render() {
        return this.props.render({
            on: this.state.on,
            toggle: this.toggle,
            reset: this.reset
            getTogglerProps: this.getTogglerProps
        });
    }
}

class App extends React.Component {

    // now you can default the state using the defaultOn prop
    render() {
        return (
            <Toggle
                defaultOn={true}
                onToggle={() => console.log('Toggle!')}
                onReset={on => console.log('Reset', on)}
                render={toggle => (
                    <div>
                        <Switch
                            ...toggle.getTogglerProps({
                                on: toggle.on
                            }) />
                        {/* we'll also add a button to reset to the initialState */}
                        <button onClick={() => toggle.reset()}>Reset</button>
                    </div>
                )} />
        );
    }
}
```
