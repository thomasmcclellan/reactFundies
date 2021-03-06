##### 5/08/2020
# Advanced Guides - Portals
Portals provide a first-class way to render children into a `DOM` node that exists outside the `DOM` hierarchy of the parent component.

```js
ReactDOM.createPortal(child, container)
```

The first argument (`child`) is any [renderable `React` child](https://reactjs.org/docs/react-component.html#render), such as an element, `string`, or fragment.  The second argument (`container`) is a `DOM` element.

---

## Usage:
Normally, when you return an element from a component's render method, it's mounted into the `DOM` as a child of the nearest parent node:

```js
render() {
  // React mounts a new div and renders the children into it
  return (
    <div>
      {this.props.children}
    </div>
  );
}
```

However, sometimes it's useful to insert a child into a different location in the `DOM`:

```js
render() {
  // React does *not* create a new div.  it renders the children into 'domNode'.
  // 'domNode' is any valid DOM node, regardless of its location in the DOM
  return ReactDOM.createPortal(
    this.props.children,
    domNode
  );
}
```

A typical use case for portals is when a parent component has an `overflow: hidden` or `z-index` style, but you need the child to visually 'break out' of its container. For example, dialogs, hovercards, and tooltips.

  > **NOTE**: When working with portals, remember that managing keyboard focus becomes very important.
  >
  > For modal dialogs, ensure that everyone can interact with them by following the `WAI-ARIA Modal Authoring Practices`.

---

## Event Bubbling Through Portals:
Even though a portal can be anywhere in the `DOM` tree, it behaves like a normal `React` child in every other way.  Features like context work exactly the same regardless  of whether the child is a portal, as the portal still exists in the _`React` tree_ regardless of position in the _`DOM` tree_.

This includes event bubbling.  An event fired from inside a portal will propagate to ancestors in the containing _`React` tree_, even if those elements are not ancestors in the _`DOM` tree_.  Assuming the following `HTML` structure:

```html
<div>
  <body id="app-root">
    <div id="modal-root">
    </div>
  </body>
</div>
```

A `Parent` component in `#app-root` would be able to catch an uncaught, bubbling event from the sibling node `#modal-root`.

```js
// These two containers are siblings in the DOM
const appRoot = document.getElementById('app-root');
const modalRoot = document.getElementById('modal-root');

class Modal extends React.Component {
  constructor(props) {
    super(props);
    this.el = document.createElement('div');
  }

  componentDidMount() {
    // The portal element is inserted in the DOM tree after
    // the Modal's children are mounted, meaning that children
    // will be mounted on a detached DOM node. If a child
    // component requires to be attached to the DOM tree
    // immediately when mounted, for example to measure a
    // DOM node, or uses 'autoFocus' in a descendant, add
    // state to Modal and only render the children when Modal
    // is inserted in the DOM tree.
    modalRoot.appendChild(this.el);
  }

  componentWillUnmount() {
    modalRoot.removeChild(this.el);
  }

  render() {
    return ReactDOM.createPortal(
      this.props.children,
      this.el
    );
  }
}

class Parent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {clicks: 0};
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    // This will fire when the button in Child is clicked,
    // updating Parent's state, even though button
    // is not direct descendant in the DOM.
    this.setState(state => ({
      clicks: state.clicks + 1
    }));
  }

  render() {
    return (
      <div onClick={this.handleClick}>
        <p>Number of clicks: {this.state.clicks}</p>
        <p>
          Open up the browser DevTools
          to observe that the button
          is not a child of the div
          with the onClick handler.
        </p>
        <Modal>
          <Child />
        </Modal>
      </div>
    );
  }
}

function Child() {
  // The click event on this button will bubble up to parent,
  // because there is no 'onClick' attribute defined
  return (
    <div className="modal">
      <button>Click</button>
    </div>
  );
}

ReactDOM.render(<Parent />, appRoot);
```

Catching an event bubbling up from a portal in a parent component allows the development of more flexible abstractions that are not inherently reliant on portals.  For example, if you render a `<Modal />` component, the parent can capture its events regardless of whether it's implemented using portals.

---

[React Docs](https://reactjs.org/docs/portals.html)