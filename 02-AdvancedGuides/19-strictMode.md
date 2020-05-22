##### 5/22/2020
# Advanced Guides - Strict Mode
`StrictMode` is a tool for highlighting potential problems in an application.  Like `Fragment`, `StrictMode` does not render any visible UI. It activates additional checks and warnings for its descendants.

  > **NOTE**: Strict mode checks are run in development mode only; _they do not impact the production build_.

You can enable strict mode for any part of your application.  For example:

```js
import React from 'react';

function ExampleApplication() {
  return (
    <div>
      <Header />
      <React.StrictMode>
        <div>
          <ComponentOne />
          <ComponentTwo />
        </div>
      </React.StrictMode>
      <Footer />
    </div>
  );
}
```

In the above example, strict mode checks will _not_ be run against the `Header` and `Footer` components.  However, `ComponentOne` and `ComponentTwo`, as well as all of their descendants, will have the checks.

`StrictMode` currently helps with:
  * Identifying components with unsafe lifecycles
  * Warning about legacy `string` ref API usage
  * Warning about deprecated `findDOMnode` usage
  * Detecting unexpected side effects
  * Detecting legacy context API

Additional functionality will be added with future releases of `React`.

## Identifying Unsafe Lifecycles:
As explained in this [blog post](https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html), certain legacy methods are unsafe for use in async `React` applications.  HOwever, if your application uses third party libraries, it can be difficult to ensure that these lifecycles aren't being used.  Fortunately, strict mode can help with this!

When strict mode is enabled, `React` compiles a list of all `class` components using the unsafe lifecycles, and logs a warning with information about these components, like so:

![Strict Mode Unsafe Lifecycles Warning](../Assets/strictModeUnsafeLifecyclesWarning.png)

Addressing the issues identified by strict mode _now_ will make it easier for you to take advantage of concurrent rendering in future releases of `React`.

## Warning About Legacy `String` Ref API Usage:
Previously, `React` provided two ways for managing refs: the legacy `string` ref API and the callback API. Although the `string` ref API was the more convenient of the two, it had [several downsides](https://github.com/facebook/react/issues/1373) and so our official recommendation was to use the callback form instead.

`React 16.3` added a third option that offers the convenience of a `string` ref without any of the downsides:

```js
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.inputRef = React.createRef();
  }

  render() {
    return <input type='text' ref={this.inputRef} />;
  }

  componentDidMount() {
    this.inputRef.current.focus();
  }
}
```

Since `object` refs were largely added as a replacement for `string` refs, strict mode now warns about usage of `string` refs.

  > **NOTE**: Callback refs will continue to be supported in addition to the new `createRef` API.
  > 
  > You don't need to replace callback refs in your components.  They are slightly more flexible, so they will remain as an advanced feature.

## Warning About Deprecated `findDOMnode` Usage:
`React` used to support `findDOMnode` to search the tree for a `DOM` node given a `class` instance.  Normally you don't need this because you can attach a ref directly to a `DOM` node.

`findDOMnode` can also be used on `class` components but this was breaking abstraction levels by allowing a parent to demand that certain children was rendered.  It creates a refactoring hazard where you can't change the implementation details of a component because a parent might be reaching into its `DOM` node.  `finedDOMnode` only returns the first child, but with the use of `Fragments`, it si possible for a component to render multiple `DOM` nodes.  `findDOMnode` is a one time read API.  It only gave you an answer when you asked for it.  If a child component renders a different node, there is no way to handle this change.  Therefore `findDOMnode` only worked if components always return a single `DOM` node that never changes.

You can instead make this explicit by passing a ref to your custom component and pass that along to the `DOM` using ref forwarding.

You can also add a wrapper `DOM` node in your component and attach a ref directly to it.

```js
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.wrapper = React.createRef();
  }

  render() {
    return <div ref={this.wrapper}>{this.props.children}</div>;
  }
}
```

  > **NOTE**: In `CSS`, the `display: contents` attribute can be used if you don't want the node to be part of the layout.

## Detecting Unexpected Side Effects:
Conceptually, `React` does work in two phases:
  1. The **render** phase determines what changes need to be made to e.g. the `DOM`.  During this phase, `React` calls `render` and then compares the result to the previous render.
  2. The **commit** phase is when `React` applies any changes (in the case of `React DOM`, this is when `React` inserts, updates and removes `DOM` nodes).  `React` also calls lifecycles like `componentDidMount` and `componentDidUpdate` during this phase.

The commit phase is usually very fast, but rendering can be slow.  For this reason, the upcoming concurrent mode (which is not enabled by default yet) breaks the rendering work into pieces, pausing and resuming the wok to avoid blocking the browser.  This means that `React` may invoke render lifecycles more than once before committing, or it may invoke them without committing at all (because of an error or a higher priority interruption).

Render phase lifecycles include the following `class` component methods:
  * `constructor`
  * `componentWillMount` (or `UNSAFE_componentWillMount`)
  * `componentWillReceiveProps` (or `UNSAFE_componentWillReceiveProps`)
  * `componentWillUpdate` (or `UNSAFE_componentWillUpdate`)
  * `getDerivedStateFromProps`
  * `shouldComponentUpdate`
  * `render`
  * `setState` updater `functions` (the first argument)

Because the above methods might be called more than once, it’s important that they do not contain side-effects. Ignoring this rule can lead to a variety of problems, including memory leaks and invalid application state. Unfortunately, it can be difficult to detect these problems as they can often be non-deterministic.

Strict mode can’t automatically detect side effects for you, but it can help you spot them by making them a little more deterministic. This is done by intentionally double-invoking the following functions:
  * `Class` component `constructor`, `render`, and `shouldComponentUpdate` methods
  * `Class` component static `getDerivedStateFromProps` method
  * `Function` component bodies
  * State updater `functions` (the first argument to `setState`)
  * `Functions` passed to `useState`, `useMemo`, or `useReducer`

  > **NOTE**: This only applies to development mode.  _Lifecycles will not be double-invoked in production mode_.

For example, consider the following code:

```js
class TopLevelRoute extends React.Component {
  constructor(props) {
    super(props);

    SharedApplicationState.recordEvent('ExampleComponent');
  }
}
```

At first glance, this code might not seem problematic. But if `SharedApplicationState.recordEvent` is not [idempotent](https://en.wikipedia.org/wiki/Idempotence#Computer_science_meaning), then instantiating this component multiple times could lead to invalid application state. This sort of subtle bug might not manifest during development, or it might do so inconsistently and so be overlooked.

By intentionally double-invoking methods like the component constructor, strict mode makes patterns like this easier to spot.

## Detecting Legacy Context API:
The legacy context API is error-prone, and will be removed in a future major version. It still works for all 16.x releases but will show this warning message in strict mode:

![Warn Legacy Context In Strict Mode](../Assets/warnLegacyContextInStrictMode.png)

---

[React Docs](https://reactjs.org/docs/strict-mode.html)