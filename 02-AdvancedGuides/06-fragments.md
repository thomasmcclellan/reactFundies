##### 4/27/2020
# Advanced Guides - Fragments
A common pattern in `React` is for a component to return multiple elements.  Fragments let you group a list of children without adding extra nodes to the `DOM`.

```js
render() {
  return (
    <React.Fragment>
      <ChildA />
      <ChildB />
      <ChildC />
    </React.Fragment>
  );
}
```

There is also a new [short syntax](https://reactjs.org/docs/fragments.html#short-syntax) for declaring them.

---

## Motivation:
A common pattern is for a component to return a list of children.  Take this example `React` snippet:

```js
class Table extends React.Component {
  render() {
    return (
      <table>
        <tr>
          <Columns />
        </tr>
      </table>
    );
  }
}
```

`<Columns />` would need to return multiple `<td>` elements in order for the rendered `HTML` to be valid.  If the parent `div` was used inside the `render()` of `<Columns />`, then the resulting `HTML` would be invalid.

```js
class Columns extends React.Component {
  render() {
    return (
      <div>
        <td>Hello</td>
        <td>World</td>
      </div>
    );
  }
}
```

results in a `<Table />` output of:

```js
<table>
  <tr>
    <div>
      <td>Hello</td>
      <td>World</td>
    </div>
  </tr>
</table>
```

Fragments solve this problem.

---

## Usage:
```js
class Columns extends React.Component {
  render() {
    return (
      <React.Fragment>
        <td>Hello</td>
        <td>World</td>
      </React.Fragment>
    );
  }
}
```

which results in a correct `<Table />` output of:

```js
<table>
  <tr>
    <td>Hello</td>
    <td>World</td>
  </tr>
</table>
```

## Short Syntax:
There is a new, shorter syntax you can use for declaring fragments.  It looks like empty tags:

```js
class Columns extends React.Component {
  render() {
    return (
      <>
        <td>Hello</td>
        <td>World</td>
      </>
    );
  }
}
```

You can use `<></>` the same way you'd use any other element except that it doesn't support keys or attributes.

## Keyed Fragments:
Fragments declared with the explicit `<React.Fragment>` syntax may have keys.  A use case for this is mapping a collection to an `array` of fragments--for example, to create a description list:

```js
function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        // Without the key, React will fire a key warning
        <React.Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </React.Fragment>
      ))}
    <dl>
  );
}
```

`key` is the only attribute that can be passed to `Fragment`.  In the future, we may add support for additional attributes, such as event handlers.

---

[React Docs](https://reactjs.org/docs/fragments.html)