## Use Render Props with React

The concept of render props in React involves passing the contents of our `render` method as a `prop` to our child components. For example:

```javascript
/* our function for rendering a Switch component. we'll use this function
in our App component, which will pass it as a prop to <Toggle />. */

function renderSwitch({ on, toggle }) {

    return <Switch on={on} onClick={toggle}>;
}

/* <Toggle /> just calls this.props.renderSwitch to render the Switch,
passing along state and static properties as args */

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
        return this.props.renderSwitch({
            on: this.state.on,
            toggle: this.toggle
        });
    }
}

// pass renderSwitch as a prop to <Toggle />
class App extends React.Component {

    render() {

        return <Toggle renderSwitch={renderSwitch}>;
    }
}
```

You could pass any function that returns valid JSX in the `renderSwitch` prop, even an arrow function. For example:

```javascript
class App extends React.Component {

    render() {

        return <Toggle renderSwitch={({ on, toggle }) => (
            <div>
                <Switch on={on} toggle={toggle}>
                {on ? 'on' : 'off}
            </div>
        )}>
    }
}
```

This pattern is called **render props** and presents several advantages over Higher Order Components. With HOCs, all child components have to be wrapped by the HOC resulting in new wrapped components. This leads to problems with things like `displayName`, nesting components while using `this.props.children`, passing of `context`, and hoisting of non-React `static` properties. It also introduces an increased chance of `prop` namespace clashes, where `props` namespaced by the HOC can overwrite, or be overwritten, by `props` passed to the child. The children give no indication if the `prop` they receive are passed through or being added by the HOC wrapper. This forces us to look at the implementation of the HOC to determine how and where `props` are passed. Render props allow us to see the `props` being applied directly by the parent, so we can use process of elimination to determine which `props` are passed by the HOC.

Composition also varies between these two approaches. HOC composition occurs statically before `render` is called – we wrap our child components in HOCs, and then we render them. With **render props**, the composition happens dynamically in the `render` method, meaning it responds elegantly and simply to changes in `props` and `state`.
