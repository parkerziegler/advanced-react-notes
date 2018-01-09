## Handle `ref` props with Higher Order Components

Be mindful that stateless functional components **cannot be given `ref`s**. This can be difficult when working with HOCS. React **does not** forward `ref`s to components wrapped by HOCs. A way to get around this is by using a secondary `prop` that holds the originally passed `ref` – something like `innerRef`. By passing this along, we obtain a `ref` for our wrapped component, which we can use to call methods, update `state`, etc. For example:

```javascript
function withToggle(Component) {

    // pass a prop called innerRef, along with other props to our HOC
    function Wrapper({ innerRef, ...props }, context) {
        const toggleContext = context['TOGGLE_CONTEXT'];

        // use innerRef to grab the instance of <Component />
        return <Component ref={innerRef} {...props} {...toggleContext}>;
    }

    return Wrapper;
}

// render our wrapped component like so
<MyToggleWrapper
    innerRef={myToggle => this.myToggle = myToggle}
/>
```
