## Make Compound React Components Flexible

The compound component strategy shown in the **[compound components example](CompoundComponents.md)** works, but it doesn't give us structural flexibility because `this.props.children` only goes one node deep. Wrapping any of the children we want to act on in a `<div>`, for example, would break the passage of implicit `state` to the intended `children`. To solve this, we can use React's [`context` API](https://reactjs.org/docs/context.html). For example:

```javascript
// import the prop-types library
import PropTypes from 'prop-types';

class Toggle extends React.Component {

    /* ensure this does not collide with any other namespaces. name it
    something unique that likely would not be used by props. */
    static TOGGLE_CONTEXT = "__toggle__"

    // set the childContextTypes
    static childContextTypes = {
        [TOGGLE_CONTEXT]: PropTypes.object.isRequired
    }

    // getChildContext() defines what gets passed in the context object
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

// declare that the component requires context
ToggleOn.contextTypes = {
    [TOGGLE_CONTEXT]: PropTypes.object.isRequired
}
```
