## Improve Unit Testability of Higher Order Components

Other developers using your HOC may want to gain access to the component they are wrapping without needing to `export` that component from the module where it's defined. We can make this easy by specifying the component passed to the HOC as a separate property on the HOC. For example:

```javascript
function withToggle(Component) {

    function Wrapper({ innerRef, ...props }, context) {
        const toggleContext = context['TOGGLE_CONTEXT'];

        return <Component ref={innerRef} {...props} {...toggleContext}>;
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
