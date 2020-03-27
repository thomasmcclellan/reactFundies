##### 3/24/2020
# Main Concepts - Introducing `JSX`
Consider this variable declaration:

```js
const element = <h1>Hello, world</h1>;
```

This funny tag syntax is neither a `string`, nor `HTML`.

It is called `JSX`, and it is a syntax extension to `JS`.  We recommend using it with `React` to describe what the UI should look like.  `JSX` may remind you of a template language, but it comes with the full power of `JS`.

`JSX` produces `React` 'elements'.  We will explore rendering them to the `DOM` in teh next section.  Below, you can find the basics of `JSX` necessary to get you started.

---

## Why `JSX`?:
`React` renders the fact that rendering logic is inherently coupled with other UI logic:  how events are handled, how the state changes over time, adn how the data is prepared for display.

Instead of artificially separating _technologies_ by putting markup and logic in separate files, `React` separates _concerns_ with loosely coupled units called 'components' that contain both. We will come back to components in further sections, but if you're not yet comfortable putting markup in `JS`, this talk might help.

`React` doesn't require using `JSX`, but most people find it helpful as a visual aid when working with UI instead of `JS` code.  It also allows `React` to show more useful error and warning messages.

With that out of the way, let's get started!

---

## Embedding Expressions in `JSX`:
In the example below, we declare a variable called `name` and then use it inside `JSX` by wrapping it in curly braces.

```js
const name = 'Tom McClellan',
      element = <h1>Hello, {name}</h1>;

ReactDOM.render(
  element,
  document.getElementById('root')
);
```

You can put any valid `JS` expression inside the curly braces in `JSX`.  For example, `2 + 2`, `user.firstName`, or `formatName(user)` are all valid `JS` expressions.

In the example below, we embed the result calling a `JS` `function`, `formatName(user)` into an `<h1>` element.

```js
function formatName(user) {
  return `${user.firstName} ${user.lastName}`;
}

const user = {
  firstName: 'Tom',
  lastName: 'McClellan'
};

const element = {
  <h1>Hello, {formatName(user)}</h1>
};

ReactDOM.render(
  element, 
  document.getElementById('root')
);
```

---

## `JSX` Is An Expression Too:
After compilation, `JSX` expressions become regular `JS` `function` calls and evaluate to `JS` `objects`.

This means that you can use `JSX` inside of `if` statements and `for` loops, assign it to variables, accept it as arguments, and return it from `functions`:

```js
function getGreeting(user) {
  if (user) return <h1>Hello, {formatName(user)}</h1>

  return <h1>Hello, stranger</h1>
}
```

---

## Specifying Attributes with `JSX`:
You may use quotes to specify `string` literals as attributes:

```js
const element = <div tabIndex="0"></div>
```

You may also use curly braces to embed a `JS` expression in an attribute:

```js
const element = <img src={user.avatarUrl} />
```

Don't put quotes around curly braces when embedding a `JS` expression in an attribute.  You should either use quotes (for `string` values) or curly braces (for expressions), but not both in the same attribute.

  > **WARNING**: Since `JSX` is closer to `JS` than to `HTML`, `React` `DOM` uses _camelCase_ property naming convention instead of `HTML` attribute names.
  >
  > For example, `class` becomes `className` in `JSX`, and `tabindex` becomes `tabIndex`.

---

## Specifying Children with `JSX`:
If a tag is empty, you may close it immediately with `/>`, like `XML`:

```js
const element = <img src={user.avatarUrl} />;
```

`JSX` tags may contain children:

```js
const element = (
  <div>
    <h1>Hello</h1>
    <h2>Good to see you here</h2>
  </div>
);
```

---

## `JSX` Prevents Injection Attacks:
It is safe to embed user input in `JSX`:

```js
const title = response.potentiallyMaliciousInput;
//This is safe:
const element = <h1>{title}</h1>;
```

By default, `React` `DOM` _escapes_ any values embedded in `JSX` before rendering them.  Thus it ensures that you can never inject anything that's not explicitly written in your application.  Everything is converted to a `string` before being rendered.  This helps prevent `XSS` (cross-site-scripting) attacks.

---

## `JSX` Represents `Objects`:
**Babel** compiles `JSX` down to `React.createElement()` calls.

These two examples are identical:

```js
const element = (
  <h1 className="greeting">
    Hello, world
  </h1>
);
```

```js
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world'
);
```

`React.createElement()` performs a few checks to help you write bug-free code but essentially it creates an `object` like this:

```js
// Note: this structure is simplified 
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world
  }
};
```

These `objects` are called '`React` elements'.  You can think of them as descriptions of what you want to see on the screen.  `React` reads these `objects` and uses them to construct the `DOM` and keep it up to date.

We will explore rendering `React` elements to the `DOM` in the next section.

---

[React Docs](https://reactjs.org/docs/introducing-jsx.html)