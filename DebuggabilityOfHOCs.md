## Improve Debuggability of Higher Order Components

You can use the `displayName` prop to alter the display name of your component in React DevTools. For example:

> App.displayName = "MyApp"; // this will render <MyApp> in DevTools

However, moments do arise where using HOCs results in `Unknown` component names in your app, especially if the HOC wraps a stateless functional component created with an arrow function.

```javascript
/* this will show up as <Unknown> in the component tree because its name
cannot be inferred from the anonymous arrow function. */
const MyToggle = withToggle(({ toggle: { on, toggle } }) => (
  <button onClick={toggle}>{on ? "on" : "off"}</button>
));
```

To fix this, it's best to specify your component separately from the instance where it gets wrapped by the HOC.

```javascript
// component can infer MyToggle as the component name
const MyToggle = ({ toggle: { on, toggle } }) => (
  <button onClick={toggle}>{on ? "on" : "off"}</button>
);

const MyToggleWrapper = withToggle(MyToggle);

// <MyToggle /> will now show up in the component tree
```

You can also make the `displayName` of your HOC explicit to show which HOC is rendering it. For example:

> Wrapper.displayName = `withToggle(${Component.displayName || Component.name})`
