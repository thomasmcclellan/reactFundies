##### 4/01/2020
# Main Concepts - Lists and Keys
First, let's review how you transform lists in `JS`.

Given the code below, we use the `map()` method to take an `array` of `numbers and double their values.  We assign the new `array` returned by `map()` to the variable `doubled` and log it.

```js
const numbers = [1, 2, 3, 4, 5],
      doubled = numbers.map(number => number * 2);

console.log(doubled);
```

This code logs `[2, 4, 6, 8, 10]` to the console.

In `React`, transforming `arrays` into lists of elements is nearly identical.

---


## Rendering Multiple Components:
You can build collections of elements and include them in `JSX` using curly braces.

Below, we loop through the `numbers` `array` using the `JS` `map()` method.  We return a `<li>` element for each of them.  Finally, we assign the resulting `array` of elements to `listItem`:

```js
const numbers = [1, 2, 3, 4, 5],
      doubled = numbers.map(number => 
        <li>{number}</li>
      );
```

We include the entire `listItem` `array` inside a `<ul>` element, and render it to the `DOM`:

```js
ReactDOM.render(
  <ul>{listItem}</ul>,
  document.getElementById('root')
);
```

This code displays a bullet list of numbers between 1 adn 5.

---

## Basic List Component:
Usually you would render lists inside a component.

We can refactor the previous example into a component that accepts an `array` of numbers and outputs a list of elements.

```js
function NumberList(props) {
  const numbers = props.numbers,
        listItems = numbers.map(number => 
          <li>{number}</li>
        );
  
  return <ul>{listItem}</ul>;
}

const numbers = [1, 2, 3, 4, 5];

ReactDOM.render(
  <NumberList number={numbers}>,
  document.getElementById('root')
);
```

When you run this code, you'll be given a warning that a key should be provided for list items.  A 'key' is a special `string` attribute you need to include when creating lists of elements.  We'll discuss why it's important in the next section.

Let's assign a `key` to our list items inside `numbers.map()` and fix the missing key issue.

```js
function NumberList(props) {
  const numbers = props.numbers,
        listItems = numbers.map(number => 
          <li key={number.toString()}>{number}</li>
        );
  
  return <ul>{listItem}</ul>;
}

const numbers = [1, 2, 3, 4, 5];

ReactDOM.render(
  <NumberList number={numbers}>,
  document.getElementById('root')
);
```

---

## Keys:
Keys help `React` identify which items have chaged, are added, or are removed.  Keys should be given to the elements inside the `array` to give the elements a stable identity:

```js
const numbers = [1, 2, 3, 4, 5],
      listItems = numbers.map(number => 
        <li key={number.toString()}>{number}</li>
      );
```

The best way to pick a key is to use a `string` that uniquely identifies a list item among its siblings.  Most often you would use IDs from your data as keys:

```js
const todoItems = todos.map(todo =>
  <li key={todo.id}>{todo.text}</li>
);
```

When you don't have stable IDs for rendered items, you may use the item index as a key as a last resort.

```js
const todoItems = todos.map((todo, index) => 
  <li key={index}>{todo.text}</li> // only do this if items have no stable IDs
);
```

We don't recommend using indexes for keys if the order of items may change.  This can negatively impact performance and may cause issues with component state.  Check out Robin Pokorny's article for an [in-depth explanation of the negative impacts of using an index as a key](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318).  If you choose not to assign an explicit key to list items then `React` will default using indexes as keys.

---

## Extracting Components with Keys:
Keys only make sense in the context of the surrounding `array`.

For example, if you _extract_ a `ListItem` component, you should keep the key on the `<ListItem>` elements in the `array` rather than on the `<li>` elemetn in the `ListItem` itself.

### Example: Incorrect Key Usage:
```js
function ListItem(props) {
  const value = props.value;
  return (
    // Wrong!  There is no need to specify the key here:
    <li key={value.toString()}>
      {value}
    </li>
  );
}

function NumberList(props) {
  const numbers = props.numbers,
        listItems = numbers.map(number => 
          // Wrong!  The key should have been specified here:
          <ListItem value={number}>
        );
  
  return (
    <ul>
      {listItem}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];

ReactDOM.render(
  <NumberList number={numbers}>,
  document.getElementById('root')
);
```

### Example: Correct Key Usage:
```js
function ListItem(props) {
  // Correct! There is no need to specify the key here:
  return <li>{props.value}</li>;
}

function NumberList(props) {
  const numbers = props.numbers,
        listItems = numbers.map(number => 
          // Correct!  The key should be specified inside the array
          <ListItem key={number.toString()} value={number}>
        );
  
  return (
    <ul>
      {listItem}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];

ReactDOM.render(
  <NumberList number={numbers}>,
  document.getElementById('root')
);
```

A good rule of thumb is that elements inside the `map()` call need keys.

---

## Keys Must Only Be Unique Among Siblings:
Keys used within `arrays` should be unique among their siblings.  However they don't need to be globally unique.  We can use the same keys when we produce two different `arrays`:

```js
function Blog(props) {
  const sidebar = (
    <ul>
      {props.posts.map(props => 
        <li key={post.id}>
          {post.title}
        </li>
      )}
    </ul>
  ),
        content = props.posts.map(post =>
          <div key={post.id}>
            <h3>{post.title}</h3>
            <p>{post.content}</p>
          </div>
        );

  return (
    <div>
      {sidebar}
      <hr>
      {content}
    </div>
  );
}

const posts = [
  {
    id: 1,
    title: 'Hello world',
    content: 'Welcome to learning React!'
  },
  {
    id: 2,
    title: 'Installation',
    content: 'You can install React from npm'
  }
];

ReactDOM.render(
  <Blog posts={posts}>,
  document.getElementById('root')
);
```

Keys serve as a hint to `React` but they don't get passed to your components.  If you need the same value in your component, pass it explicitly as a prop with a different name:

```js
const content = posts.map(post =>
  <Post
    key={post.id}
    id={post.id}
    title={post.title}
  >
);
```

With the example above, the `Post` component can read `props.id`, but not `props.key`.

---

## Embedded `map()` In `JSX`:
In the examples above we declared a separate `listItem` variable and included it in `JSX`:

```js
function NumberList(props) {
  const numbers = props.numbers,
        listItems = numbers.map(number =>
          <ListItem key={number.toString()} value={number}>
        );
  
  return (
    <ul>
      {listItem}
    </ul>
  );
}
```

`JSX` allows embedding any expression in curly braces so we could inline the `map()` result:

```js
function NumberList(props) {
  const numbers = props.numbers;

  return (
    <ul>
      {numbers.map(number => 
        <ListItem key={number.toString()} value={number}>
      )}
    </ul>
  );
}
```

Sometimes this results in clearer code, but this style can also be abused.  Like in `JS`, it is up to you to decide whether it is worth extracting a variable for readability.  Keep in mind that if the `map()` body is too nested, it might be a good time to extract a component.

---

[React Docs](https://reactjs.org/docs/lists-and-keys.html)