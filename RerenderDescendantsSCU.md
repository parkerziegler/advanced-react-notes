## Rerender Descendants Through `shouldComponentUpdate`

A `Provider` is intended to give components access to `state` from anywhere in the application. A part of the React component lifecycle can interfere with this however. That part is `shouldComponentUpdate`. `shouldComponentUpdate` is a performance optimization that prevents components from re-rendering when there is nothing to update. While `shouldComponentUpdate` is implemented for you under the hood, you can implement it yourself to win some performance improvements.

The issue with `shouldComponentUpdate` is that any descendants of a component where `shouldComponentUpdate` returns `false` **will not be rerendered**. We need a way to inform these components of `state` changes even when their parent does not update. [`react-broadcast`](https://github.com/ReactTraining/react-broadcast) can help us solve this problem. Let's see how we might reimplement our `ToggleProvider` component with `react-broadcast`.

```javascript
class ToggleProvider extends React.Component {
    static channel = "__toggle_channel__"
    render() {
        const {
            children,
            ...remainingProps
        } = this.props;

        return (
            <Toggle {...remainingProps}
                render={toggle => (
                    <ReactBroadcast.Broadcast
                        channel={ToggleProvider.channel}
                        value={toggle}
                    >
                        {children}
                    </ReactBroadcast.Broadcast>
                )}
            >
        )
    }
}
```

Now, any time the `toggle` state changes, `React.Broadcast` will be notified and will notify all subscribers. Now we just need to ensure that components that need access to the channel are subscribed to it.

```javascript
function ConnectedToggle(props, context) {
  return (
    <ReactBroadcast.Subscriber channel={ToggleProvider.channel}>
      {toggle => props.render(toggle)}
    </ReactBroadcast.Subscriber>
  );
}
```

This allows us to nest child components in a parent component where `shouldComponentUpdate` evaluates to `false`. `ConnectedToggle` now provides access to a subscriber, which calls our `render prop` with the value of `toggle` whenever a `state` change is received over the channel.
