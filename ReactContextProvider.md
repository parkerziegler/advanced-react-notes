## Implement a React Context Provider

The **Provider pattern** in React helps to solve a classic problem – prop drilling. Prop drilling occurs when we forward props several levels deep, resulting in difficult to maintain code. Components may not even use these props, their responsibility being to solely pass the props along to children. We can fix this by creating a **Provider component** that holds `context`. To access this `context`, we simply wrap components in a **connector component**, which uses the **render props** pattern to render `children` with `context` provided. In the example below, `<ToggleProvider />` is our **Provider component** and `<ConnectedToggle />` is our **connector component**. For example:

```javascript
/* the ToggleProvider component holds the context that we
want to share throughout our component tree. It provides context
to this.props.children to help share implicit state. */
class ToggleProvider extends React.Component {

    static contextName = "__toggle__"

    /* the Renderer component here just returns this.props.chidlren
    and specifies our getChildContext() method to actually describe what
    gets placed on the context object */
    static Renderer = class extends React.Component {

        static childContextTypes = {
            [ToggleProvider.contextName]: PropTypes.object.isRequired
        }

        getChildContext() {
            return {
                [ToggleProvider.contextName]: this.props.toggle
            }
        }

        render() {
            return this.props.children;
        }
    }

    render() {
        const { children, ...rest } = this.props;
        return (
            <Toggle {...rest} render={toggle => (
                <ToggleProvider.Renderer
                    toggle={toggle}
                    children={children} />
            )} />
        )
    }
}

function ConnectedToggle(props, context) {
    // return the render prop, passing in the context as arg
    return props.render(context[ToggleProvider.contextName]);
}

ConnectedToggle.contextTypes = {
    [ToggleProvider.contextName]: PropTypes.object.isRequired
}

function Article() {
    return (
        <div>
            <ConnectedToggle render={({ toggle }) => (
                /* any component we want to render using render props
                now has access to toggle via context! */
            )} />
            <div>Some sort of chatter and discussion woohoo!<div>
        </div>
    )
}

function App() {

    {/* we don't need to pass any props to <Article /> or in this case.
    <ToggleProvider /> provides the necessary context to these components, so they
    have access to implicit state. */}
    return (
        <ToggleProvider>
            <Article />
        </ToggleProvider>
    );
}
```
