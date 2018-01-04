# Advanced React Component Patterns - Kent C. Dodds

## Compound Components
**Compound components** are a technique used to share implicit `state` between a set of components. A top-level component wraps one or more child components, with each component in the tree gaining access to the `state` of the top-level component. To accomplish this, we use `React.Children.map`, a special method for iterating over React children. This allows us to pass elements of the parent's `state` (or static properties, callbacks, etc.) as `props` to all `children`. For example:

```javascript

// render of top-level component
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
```

## Make Compound React Components Flexible
The above strategy works, but it doesn't give us structural flexibility because `this.props.children` only goes one node deep. Wrapping any of the children we want to act on in a `<div>`, for example, would break the passage of implicit `state` to the intended `children`. To solve this, we can use React's [`context` API](https://reactjs.org/docs/context.html). For example:

```javascript
// parent component
class Toggle extends React.Component {

    // ensure this does not collide with any other namespaces
    static TOGGLE_CONTEXT = "__toggle__"

    // set the childContextTypes
    static childContextTypes = {
        [TOGGLE_CONTEXT]: PropTypes.object.isRequired
    }

    // this method defines what gets passed in the context object
    getChildContext() {

        return {
            on: this.state.on,
            toggle: this.toggle
        };
    }

    render() {

        /* we no longer need to use React.Children.map because all children
        are already passed the data they need through context */
        return <div>{this.props.children}</div>;
    }
}

// a child with access to context
function ToggleOn = ({ children }, context) {
    const { on } = context['TOGGLE_CONTEXT']
    return on ? children : null
}

// this declares that the component requires context
ToggleOn.contextTypes = {
    [TOGGLE_CONTEXT]: PropTypes.object.isRequired
}
```

## Make Enhanced React Components With Higher Order Components
**Higher Order Components** (HOCs) in React are function factories that accept a component as input and return a component as output. Typically, they add some abstract functionality to the component (via `props`) that is useful. For example:

```javascript
function withToggle(Component) {

    /* wrapper will forward along all the props and context to whatever component
    gets passed in to the withToggle HOC. */
    function Wrapper(props, context) {

        const toggleContext = context['TOGGLE_CONTEXT'];

        return <Component {...props} {...toggleContext}>;
    }

    ToggleOn.contextTypes = {
        [TOGGLE_CONTEXT]: PropTypes.object.isRequired
    }

    return Wrapper;
}
```

## Handle `prop` Namespace Clashes with Higher Order Components
Keep in mind that there can be **namespace clashes** when using HOCs, especially if the HOC passes a `prop` or a piece of `context` with the same name of a `prop` or piece of `state` that the parent passes to the wrapped child component. **This can lead to unexpected behavior**. For example:

```javascript
function MyChildComponent(props) {

    return <div>{props.on}</div>;
}

function MyNewComponent() {

    // this wrapped component will receive an on prop and an on context value
    return withToggle(MyChildComponent({ on }))
}

function withToggle(Component) {

    /* wrapper will forward along all the props and context to whatever component
    gets passed in to the withToggle HOC. */
    function Wrapper(props, context) {

        // context has a key-value pair for on
        const toggleContext = context['TOGGLE_CONTEXT'];

        // the toggleContext "on" will overwrite the prop "on" since it is passed later
        return <Component {...props} {...toggleContext}>;
    }

    ToggleOn.contextTypes = {
        [TOGGLE_CONTEXT]: PropTypes.object.isRequired
    }

    return Wrapper;
}
```

A way to get around this is by namespacing the `props` passed by your HOC. For example:

```javascript
/* namespace toggleContext as a toggle prop - this holds the whole context object in this prop,
which you can then document as being reserved by your HOC. */
return <Component {...props} toggle={...toggleContext}>;
```