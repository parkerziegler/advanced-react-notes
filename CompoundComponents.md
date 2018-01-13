## Compound Components

**Compound components** are a technique used to share implicit `state` (and other properties like `static`s and callbacks) between a set of components. A top-level component wraps one or more child components, with each component in the tree gaining access to specified `state`, properties, or functions of the top-level component. To accomplish this, we use `React.Children.map`, a special method for iterating over React children. This allows us to pass elements of the parent's `state` or other properties as `props` to all `children`. For example:

```javascript
class Toggle extends React.Component {

    // state initialized on Toggle
    state = {
        on: false
    }

    // callback for updating the state
    toggle = () => {
        this.setState({
            on: !this.state.on
        });
    }

    render() {

        // iterate using React.Children.map
        const children = React.Children.map(
            this.props.children,
            child =>
                /* for each child passed, clone it and provide some additional props.
                this allows us to share implicit state between all components. */
                React.cloneElement(child, {
                    on: this.state.on,
                    toggle: this.toggle
                })
        );

        return <div>{children}</div>;
    }
}

/* stateless functional components that will be rendered as children of <Toggle>.
notice they gain access to the on propety of Toggle's state as well as the toggle
callback function. */
const ToggleOn = ({ on, toggle }) => (
    on ? <div onClick={toggle}>On</div> : null;
);

const ToggleOff = ({ on, toggle }) => (
    on ? null : <div onClick={toggle}>Off</div>;
);

// render the App
class App extends React.Component {

    render() {
        <Toggle>
            <ToggleOn>
            <ToggleOff>
        </Toggle>
    }
}
```
