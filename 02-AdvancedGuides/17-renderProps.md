##### 5/18-19/2020
# Advanced Guides - Render Props
The term _render props_ refers to a technique for sharing code between `React` components using a prop whose value is a `function`.

A component with a render prop takes a `function` that returns a `React` element and calls it instead of implementing its own render logic.

```js
<DataProvider render={data => (
  <h1>Hello {data.target}</h1>
)} />
```

Libraries that use render props include `React` Router, [Downshift](https://github.com/downshift-js/downshift), [Formik](https://github.com/jaredpalmer/formik)

In this document, we'll discuss why render props are useful, and how to write your own.

---

## Use Render Props for Cross-Cutting Concerns:
Components are the primary unit of code reuse in `React`, but it's not always obvious how to share the state or behavior that one component encapsulates to other components that need that same state.

For example, the following components tracks the mouse position in a web app:

```js
class MouseTracker extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = {
      x: 0,
      y: 0
    };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div
        style={{ height: '100vh' }}
        onMouseMove={this.handleMouseMove}
      >
        <h1>Move the mouse around</h1>
        <p>The current mouse position is ({this.state.x}, {this.state.y})</p>
      </div>
    );
  }
}
```

As the cursor moves around the screen, the component displays its (x, y) coordinates in a `<p>`.

Now the question is: How can we reuse this behavior in another component? In other words, if another component needs to know about the cursor position, can we encapsulate that behavior so that we can easily share it with that component?

Since components are the basic unit of code reuse in `React`, let's try refactoring the code a bit to use a `<Mouse>` component that encapsulates the behavior we need to reuse elsewhere.

---

## Using Props Other Than `render`:
It's important to remember that just because the pattern is called _render props_ you don't _have to use a prop named render to use this patter_.  In fact, _any_ prop that is a `function` that a component uses to know what to render is technically a _render prop_.

Although the examples above use `render`, we could just as easily use the `children` props.

```js
<Mouse children={mouse => (
  <p>The mouse position is {mouse.x}, {mouse.y}</p>
)} />
```

And remember, the `children` prop doesn't actually need to be named in the list of 'attributes' in your `JSX` element.  Instead, you can put it directly _inside_ the element.

```js
<Mouse>
  {mouse => (
    <p>The mouse position is {mouse.x}, {mouse.y}</p>
  )}
</Mouse>
```

You'll see this technique used in the [`react`-motion](https://github.com/chenglou/react-motion) API.

Since this technique is a little unusual, you'll probably wan tot explicitly state that `children` should be a `function` in your `propTypes` when designing an API like this.

```js
Mouse.propTypes = {
  children: PropTypes.func.isRequired
};
```

---

## Caveats:
**Be careful when using Render Props with `React.PureComponent`**

Using a render prop can negate the advantage that comes from using [`React.PureComponent`](https://reactjs.org/docs/react-api.html#reactpurecomponent) if you create the `function` inside a render method. This is because the shallow prop comparison will always return `false` for new props, and each render in this case will generate a new value for the render prop.

For example, continuing with our `<Mouse>` component from above, if `Mouse` were to extend `React.PureComponent` instead of `React.Component`, our example would look like this:

```js
class Mouse extends React.PureComponent {
  // Same implementation as above...
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>

        {/*
          This is bad! The value of the `render` prop will
          be different on each render.
        */}
        <Mouse render={mouse => (
          <Cat mouse={mouse} />
        )}/>
      </div>
    );
  }
}
```

In this example, each time `<MouseTracker>` renders, it generates a new `function` as the value of the `<Mouse render>` prop, thus negating the effect of `<Mouse>` extending `React.PureComponent` in the first place!

To get around this problem, you can sometimes define the prop as an instance method, like so:

```js
class MouseTracker extends React.Component {
  // Defined as an instance method, `this.renderTheCat` always
  // refers to *same* function when we use it in render
  renderTheCat(mouse) {
    return <Cat mouse={mouse} />;
  }

  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <Mouse render={this.renderTheCat} />
      </div>
    );
  }
}
```

In cases where you cannot define the prop statically (e.g. because you need to close over the component’s props and/or state) `<Mouse>` should extend React.Component instead.

---

[React Docs](https://reactjs.org/docs/render-props.html)