---
title: "How React Updates State"
description: "If you're updating the state of a React component, expect the state variable to receive the new value on the next rendering."
published: "2020-12-15T12:30Z"
modified: "2020-12-15T12:30Z"
thumbnail: "./images/cover-3.png"
slug: how-react-updates-state
tags: ['react', 'hook', 'usestate']
recommended: ['react-usestate-hook-guide', 'react-hooks-mistakes-to-avoid']
type: post
---

React `useState()` hook manages the state in functional React components. In class components `this.state` holds the state, and you invoke the special method `this.setState()` to update the state.   

Mostly using state in React is straightforward. However, there's an important nuance to be aware of when updating the state.  

When you update the component's state, does React update the state *immediately* (synchronously) or rather *schedules a state update* (asynchronously)? This post answers this question.  

## 1. State update using *useState()*

Consider a functional component `DoubleIncreaser`:

```jsx{6-7}
import { useState } from 'react';

function DoubleIncreaser() {
  const [count, setCount] = useState(0);

  const doubleIncreaseHandler = () => {
    setCount(count + 1);
    setCount(count + 1);
  };

  return (
    <>
      <button onClick={doubleIncreaseHandler}>
        Double Increase
      </button>
      <div>Count: {count}</div>
    </>
  );
}
```

`const [count, setCount] = useState(0)` defines the component state. `count` is the state variable containing the current state value, while `setCount()` is the state updater function.  

The component has a button *Double Increase*. When the button is clicked, `doubleIncreaseHandler` event handler performs 2 consecutive increments of count state: `setCount(count + 1)` and then `setCount(count + 1)` again.  

When clicking *Double Increase*, does the component's state update by `1` or `2`?  

Open the [demo](https://codesandbox.io/s/usestate-broken-ytmxk?file=/src/index.js) and click the button *Double Increase*. The `count` is increased by `1` on each click.  

When `setCount(count + 1)` updates the state, the changes are not reflected immediately in the `count` variable. Rather React *schedules a state update*, and during the next rendering in the statement `const [count, setCount] = useState(0)` the hook assigns to `count` the new state value.  

For instance: if `count` variable is `0`, then calling `setCount(count + 1); setCount(count + 1); ` is simply evaluated as `setCount(0 + 1); setCount(0 + 1);` &mdash; making the state on next render as `1`.  

> The state update function `setValue(newValue)` of `[value, setValue] = useState()` updates the state asynchronously.  

The state update function also accepts a callback to compute new state using the current state. In case of the `DoubleIncreaser`, you can use `setCount(actualCount => actualCount + 1)`:  

```jsx{6-7}
import { useState } from 'react';

function DoubleIncreaser() {
  const [count, setCount] = useState(0);

  const doubleIncreaseHandler = () => {
    setCount(actualCount => actualCount + 1);
    setCount(actualCount => actualCount + 1);
  };

  return (
    <>
      <button onClick={doubleIncreaseHandler}>
        Double Increase
      </button>
      <div>Count: {count}</div>
    </>
  );
}
```

When updating the state using a function `setCount(actualCount => actualCount + 1)`, then `actualCount` argument contains the actual value of the state.  

Open the [demo](https://codesandbox.io/s/usestate-fixed-callback-e4pp3?file=/src/index.js), and click the *Double Increase* button. The count updates by `2` as expected.  

Of course, you can use an intermediate `let` variable:

```jsx{6-9}
import { useState } from 'react';

function DoubleIncreaser() {
  const [count, setCount] = useState(0);

  const doubleIncrease = () => {
    let actualCount = count;
    actualCount = actualCount + 1;
    actualCount = actualCount + 1;
    setCount(actualCount);
  };

  return (
    <>
      <button onClick={this.doubleIncrease}>
        Double Increase
      </button>
      <div>Count: {count}</div>
    </>
  );
}
```

`let actualCount = count` is an intermediate variable that you can update at will. The updated intermediate variable to update the `setCount(actualCount)`.  

Try the [demo](https://codesandbox.io/s/usestate-fixed-interm-variable-xo3n7?file=/src/index.js) using intermediate variable.  

## 2. The state variable is immutable and readonly

If you forget that the state variable updates on the next rendering, you might try to read the state variable right after changing it. Unfortunately, it won't work:  

```jsx{9-10}
function FetchUsers() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    const startFetching = async () => {
      const response = await fetch('/users');
      const fetchedUsers = await response.json();

      setUsers(fetchedUsers);
      console.log(users);        // => []
      console.log(fetchedUsers); // => ['John', 'Mike', 'Denis']
    };
    startFetching();
  }, []);

  return (
    <ul>
      {users.map(user => <li>{user}</li>)}
    </ul>
  );
}
```

`FetchUsers` component starts a fetch request on mounting &mdash; `startFetching()`.  

When fetch is completed, `setUsers(fetchedUsers)` updates the state with the fetched users. However, the changes aren't reflected right away in `users` state variable.  

The state variable `users` is readonly and immutable. Only `useState()` hook assigns values to `users`. You're not allowed manually to assign to state variable, neither mutate it:

```jsx{8-10}
function FetchUsers() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    const startFetching = async () => {
      const response = await fetch('/users');
      const fetchedUsers = await response.json();

      users = fetchedUsers;       // Incorrect! users is readonly
      users.push(..fetchedUsers); // Incorrect! users is immutable
      setUsers(fetchedUsers);     // Correct!
    };
    startFetching();
  }, []);

  return (
    <ul>
      {users.map(user => <li>{user}</li>)}
    </ul>
  );
}
```

## 3. State update in a class component

The same idea of the asynchronous state update is valid for class components too.  

The following class component has a state `count`, which should increase by `2` when the button *Double Increase* is clicked:

```jsx{20-25}
import { Component } from 'react';

class DoubleIncreaser extends Component {
  state = {
    count: 0
  };

  render() {
    return (
      <>
        <button onClick={this.doubleIncrease}>
          Double Increase
        </button>
        <div>Count: {this.state.count}</div>
      </>
    );
  }

  doubleIncrease = () => {
    // Works!
    this.setState(({ count }) => ({
      count: count + 1
    }));
    this.setState(({ count }) => ({
      count: count + 1
    }));

    // Won't work!
    // this.setState({ count: this.state.count + 1 });
    // this.setState({ count: this.state.count + 1 });
  }
}
```

Take a look at the `doubleIncrease()` event handler: the state updater also uses a callback.  

Open the [demo](https://codesandbox.io/s/thisstate-chrfo?file=/src/index.js) and click the button *Double Increase*. As expected, the `this.state.count` updates by 2. 

In class based components, `this.state` is also not updated immediately. When calling `this.setState(newState)`, React schedules a re-render, and exactly on next rendering
`this.state` contains the new state value `newState`.  

> `this.setState(newState)` updates `this.state` asynchronously.  

## 4. Summary

`useState()` hook and `this.setState()` (inside class components) update the state variable and the component output asynchronously.  

Remember the simple rule: 

> Calling the setter function `setValue(newValue)` of `useState()` hook (or `this.setState()` of class components) doesn't exactly update the state, but rather *schedules a state update*.  

*Quiz: are references (created by `useRef()`) updated synchronously or asynchronously? Write the answer in a comment below!*