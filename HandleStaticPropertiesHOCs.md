## Handle Static Properties Properly with Higher Order Components

Remember that when defining `static` properties on a component wrapped by an HOC, those properties **are not directly accessible from the wrapped component instance**. However, we would like `static` properties to persist through the process of HOC wrapping. In essence, we want to make the use of our HOC as "inobservable as possible." To do this, we want to hoist all `static` properties that are not React specific (i.e. `displayName`) so they apply to our HOC. We can use the [`hoist-non-react-statics`](https://github.com/mridgway/hoist-non-react-statics) library to do this. For example:

```javascript
// import hoist-non-react-statics
import hoistNonReactStatics from 'hoist-non-react-statics';

function withToggle(Component) {

    function Wrapper({ innerRef, ...props }, context) {
        const toggleContext = context['TOGGLE_CONTEXT'];

        return <Component ref={innerRef} {...props} {...toggleContext}>;
    }

    Wrapper.WrappedComponent = Component;

    /* here we will hoist all non-React static properties of
    Wrapper and Component. */
    return hoistNonReactStatics(Wrapper, Component);
}
```
