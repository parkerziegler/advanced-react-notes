## Handle `prop` Namespace Clashes with Higher Order Components

Keep in mind that there can be **namespace clashes** when using HOCs, especially if the HOC passes a `prop` or a piece of `context` with the same name of a `prop` or piece of `state` passed by the parent to the wrapped child component. **This can lead to unexpected behavior**. For example:

```javascript
// a child component, which receives an on prop from the parent
function MyChildComponent({ on }) {

    return <div>{on}</div>;
}

/* a parent component, which passes an on prop to a child component wrapped
by an HOC. The issue arises that withToggle passes a piece of context named "on"
to the child as a prop. This will lead to a namespace clash. */
function MyParentComponent() {

    return withToggle(MyChildComponent({ on }))
}

function withToggle(Component) {

    function Wrapper(props, context) {

        // context has a key-value pair for on
        const toggleContext = context['TOGGLE_CONTEXT'];

        /* the toggleContext "on" will overwrite the parent prop
        "on" since it is passed later */
        return <Component {...props} {...toggleContext}>;
    }

    return Wrapper;
}
```

A way to get around this is by namespacing the `props` passed by your HOC. For example:

```javascript
/* namespace toggleContext as a prop called "toggle" - the context
object is now held entirely in this prop, which you can then document
as being reserved by your HOC. */
return <Component {...props} toggle={...toggleContext}>;
```
