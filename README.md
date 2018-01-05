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

## Improve Debuggability of Higher Order Components
You can use the `displayName` prop to alter the display name of your component in React DevTools. For example:

> App.displayName = "MyApp"; // this will render <MyApp> in DevTools

However, moments do arise where using HOCs results in `Unknown` component names in your app, especially if the HOC wraps a stateless functional component created with an arrow function.

```javascript
/* this will show up as <Unknown> in the component tree because its name
cannot be inferred from the anonymous arrow function. */
const MyToggle = withToggle(
    ({ toggle: { on, toggle }}) => (
        <button onClick={toggle}>
            {on ? 'on' : 'off'}
        </button>
    )
);
```

To fix this, it's best to specify your component separately from the instance where it gets wrapped by the HOC.

```javascript
// component can infer MyToggle as the component name
const MyToggle = ({ toggle: { on, toggle }}) => (
        <button onClick={toggle}>
            {on ? 'on' : 'off'}
        </button>
    );

const MyToggleWrapper = withToggle(MyToggle);

// MyToggle will now show up in the component tree
```

You can also make the `displayName` if your HOC explicit, i.e.:

> Wrapper.displayName = `withToggle(${Component.displayName || Component.name})`

## Handle `ref` props with Higher Order Components
Be mindful that stateless functional components **cannot be given `ref`s**. This can be difficult when working with HOCS. React **does not** forward `ref`s to components wrapped by HOCs. A way to get around this is by using a secondary `prop` that holds the originally passed `ref` – something like `innerRef`. By passing this along, we obtain a `ref` for our wrapped component, which we can use to call methods, update `state`, etc.

```javascript
function withToggle(Component) {

    // pass a prop called innerRef, along with other props
    function Wrapper({ innerRef, ...props }, context) {
        const toggleContext = context['TOGGLE_CONTEXT'];

        // use innerRef to grab the instance of Component
        return <Component ref={innerRef} {...props} {...toggleContext}>;
    }

    ToggleOn.contextTypes = {
        [TOGGLE_CONTEXT]: PropTypes.object.isRequired
    }

    return Wrapper;
}

// jsx for our wrapped component
<MyToggleWrapper
    innerRef={myToggle => this.myToggle = myToggle}
/>
```

## Improve Unit Testability of Higher Order Components
Other developers using your HOC may want to gain access to the component they are wrapping without needing to `export` that component from the module where it's defined. We can make this easy by specifying the component passed to the HOC as a separate property on the HOC. For example:

```javascript
function withToggle(Component) {

    function Wrapper({ innerRef, ...props }, context) {
        const toggleContext = context['TOGGLE_CONTEXT'];

        return <Component ref={innerRef} {...props} {...toggleContext}>;
    }

    ToggleOn.contextTypes = {
        [TOGGLE_CONTEXT]: PropTypes.object.isRequired
    }

    // here we assign Component to the WrappedComponent property on Wrapper
    Wrapper.WrappedComponent = Component;
    return Wrapper;
}

// test the rendering of the wrapped component
ReactDOM.render(
    <Wrapper.WrappedComponent />,
    document.getElementById("#wrapperDiv")
);
```

## Handle Static Properties Properly with Higher Order Components
Remember that when defining `static` properties on a component wrapped by an HOC, those properties **are not directly accessible from the wrapped component instance**. However, we would like `static` properties to persist through the process of HOC wrapping. In essence, we want to make the use of our HOC as "inobservable as possible." To do this, we want to hoist all `static` properties that are not React specific (i.e. `displayName`) so they apply to our HOC. We can use the [`hoist-non-react-statics`](https://github.com/mridgway/hoist-non-react-statics) library to do this. For example:

```javascript
function withToggle(Component) {

    function Wrapper({ innerRef, ...props }, context) {
        const toggleContext = context['TOGGLE_CONTEXT'];

        return <Component ref={innerRef} {...props} {...toggleContext}>;
    }

    ToggleOn.contextTypes = {
        [TOGGLE_CONTEXT]: PropTypes.object.isRequired
    }

    Wrapper.WrappedComponent = Component;

    /* here we will hoist all non-React static properties of
    Wrapper and Component. */
    return hoistNonReactStatics(Wrapper, Component);
}
```