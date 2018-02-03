## Implement a Higher Order Component with Render Props

**Render props** can make it easy for us to work with HOCs. Rather than passing the `Component` as the direct return of our HOC factory, we can return a different component that accepts the rendering of `Component` as a render prop. In the example below, `withToggle` always renders a `ConnectedToggle` HOC, which accepts a render prop responsible for rendering the component we pass in to `withToggle`.

```javascript
function withToggle(Component) {
    function Wrapper(props, context) {
        const { innerRef, ...remainingProps } = props;
        {/* pass ConnectedToggle a render prop */}
        return <ConnectedToggle render={toggle => (
            <Component {...remainingProps} toggle={toggle} />
        )} />
    }

    Wrapper.displayName = `withToggle(${Component.displayName || Component.name})`;
    Wrapper.propTypes = { innerRef: PropTypes.func };
    Wrapper.WrappedComponent = Component;
    hoistNonReactStatics(Wrapper, Component)
}

/* ConnectedToggle accepts a render prop, which gets invoked with
the value of toggle provided by context */
function ConnectedToggle(props, context) {
    return props.render(
        context[ToggleProvider.contextName],
    )
}
ConnectedToggle.contextTypes = {
  [ToggleProvider.contextName]: PropTypes.object.isRequired
}

/* wrap Subtitle in the withToggle HOC. this will render Subtitle
along with a Toggle component */
const Subtitle = withToggle({ toggle }) => (
    toggle.on ? 'on some emojis' : 'off no emojis'
);
```
