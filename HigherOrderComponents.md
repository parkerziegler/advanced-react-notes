## Make Enhanced React Components With Higher Order Components

**Higher Order Components** (HOCs) in React are function factories that accept a component as input and return a component as output. Typically, they add some abstract functionality to the component (usually by adding `props` or `context`) that is useful. For example:

```javascript
// a higher order component that adds context to the component passed as an arg
function withToggle(Component) {

    /* wrapper will forward along all the props and context to whatever component
    gets passed in to the withToggle HOC. */
    function Wrapper(props, context) {

        const toggleContext = context['TOGGLE_CONTEXT'];

        return <Component {...props} {...toggleContext}>;
    }

    return Wrapper;
}
```
