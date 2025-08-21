+++
author = "Udara Bibile"
authorImage = "/img/udarabibile.jpg"
title = "Introduction to React Hooks"
date = "2019-07-19"
description = "Giving superpowers to functional components"
tags = ["reactjs", "syntax"]
categories = ["javascript", "reactjs"]
images  = ["img/2019/react-hooks.png"]
type = "post"
aliases = ["migrate-from-jekyl"]
draft = false
+++

From **React’s version 16.8.0**, Hooks are introduced to make functional components more useful.

<figure>
  <img class="image featured" src="/img/2019/class-functional-component.png" alt="" style="width:100%" >
  <figcaption style="text-align:center">React: Class Component vs Functional Component.</figcaption>
</figure>

<mark>TODO: MAKE MORE WIDTH</mark>

In React, **class components** are **stateful and smart**, being state is attached to it and kept persistent through renders. Then **functional components** were **stateless and dumb** components, having nor state nor logic attached to it.

As an example, class component can have `state`, and it could be manipulated using lifecycle methods such as `constructor`, `componentDidMount` and such. However, functional component only accepts `props` from its parent, and renders accordingly. It wouldn’t keep internal `state` value, that would be persistent through renders, but rather renders according to whatever passed through `props`.

In complex React application, there is parent *class component* keeping track of state, and manipulate state according to business logic. Afterwards child *functional components* are passed props, and used only for rendering purpose. These wouldn’t have any persistent state in it, and just used to break down complex components into segments.

```
<DigitalWatch>
Parent class component with state, and business logic to count time.

→ <TimeView time={this.state.time}>
Child functional component only with props and just rendering props.
```
<hr/>

React Hooks enable **functional components to attach local state with it, and React will preserve this state between re-renders**. Thereby this will allow React to be used without classes.

> However, we often can’t break complex components down any further because the logic is stateful and can’t be extracted to a function or another component.</p>
> — <cite>Dan Abramov[^1]</cite>

[^1]: Extracted from [medium article](https://medium.com/@dan_abramov/making-sense-of-react-hooks-fdbde8803889).

<br/>
As given, it’ll reduce usage of complex class components with all logic in it. Rather some of this business logic will be passed to its child functional component. This refactored functional components will have its functionality attached with it, so would act as reusable small components.

**Let's get started with hooks**: (Note that hooks doesn’t work inside classes.)

<hr/>

### State Hook

```
import React, { useState } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}> Add </button>
      <button onClick={() => setCount(count - 1)}> Remove </button>
    </div>
  );
}
```

State hook here, just introduce **`this.State`** and **`this.setState`** to functional components. This `useState` returns value, its accessor and is initiated at the start: **`const [state, setState] = useState(initialState);`**. Thereby in retrospective, in contrast to class components by `Constructor`, and state manipulated functions such as *handleIncrease* which was bind to `this` scope.

Moreover state hook can be initiated for many types:

```
const [num, setNum] = useState(10);
const [name, setName] = useState('default');
const [obj, setObj] = useState([{ text: 'this is object' }]);
```

<hr/>

### Effect Hook

Effect hook here, is used to perform side effects in functional components. From familiar user cases, it could be considered combination of methods: **`componentDidMount`**, **`componentDidUpdate`**, and **`componentWillUnmount`**. So when React renders (updates the DOM including first render) component, this hook becomes applied. Typical use cases for effect hooks are data fetching, managing subscriptions and dealing with manual DOM changes.

#### Effect Hook without cleanup

Here at each time this functional component renders (including first render and subsequent re-renders due to `count` change) will execute effect hook function. In this instance it’ll perform side effect of manual DOM change by manipulating external element.

```
import React, { useState, useEffect } from 'react';

function App() {
  const [count, setCount] = useState(0);

  useEffect(() => {
     document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}> Click </button>
    </div>
  );
}
```

#### Conditional execution of effect hook

This effect hook acts for **each render**, so can replace lifecycle methods such as **`componentDidMount`** & **`componentDidUpdate`**. So in above example, hook function will execute on **component mounting (first render) as well as component updating (re-rendering)**, and might not be required to execute every time. Thereby effect hook execution can be restricted, and decided by **second parameter** passed to **`useFffect`**.

```
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]);
```

In above hook, it’ll be executed at first render and whenever there is change in `count` as . given by second parameter as `[count]`. Here hook is only useful as long as there is a change to that state value. Moreover if `[]` is given as parameter, it’ll only be executed as mounting, as there this value is independent of state and will not change with re-renders.

##### Data fetching in effect hook

In class components, data fetching was done in lifecycle methods, and therefore in functional components it’ll be done in hook. As learned above, optional parameter is added so that data fetching happens only in mounting.

```
import React, { useState, useEffect } from 'react';

const App = () => {
  const [data, setData] = useState({ arr: [], int: 10 });
  useEffect(() => {
     async function fetchData() {
        const response = await fetchMockAPI();
        setData({...data, arr: response});
     }

     fetchData();
  }, []);

return (
    <h1>Arr: {data.arr.length} & Int: {data.int}</h1>
  );
}

export default App;

const fetchMockAPI = async() =>
   new Promise(resolve => setTimeout(resolve([1,2]), 1000));
```

#### Effect Hook with cleanup

Suppose in class component, you’ll be subscripting to external source in **`componentDidMount`**. Thereby when removing the component, subscription should be removed in **`componentWillUnmount`**. In hooks it's called **cleanup**, and if function is returned from effect hook (optional) it’ll be used for cleanup when unmounting this functional component.

```
import React, { useState, useEffect } from 'react';

function FriendStatus(id) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribe(id, handleStatusChange);
    return function cleanup() {
      ChatAPI.unsubscribe(id, handleStatusChange);
    };
  });
  
  return(
     if (isOnline === null) 'Loading...';
     else isOnline ? 'Online' : 'Offline';
  );
}
```

Moreover, it is practical to use multiple effect hooks for separation of concern and to deal with multiple state values.

<hr/>

### Context Hook

This is applicable when using **Context API** introduced in React 16.3.0 to avoid props drilling. Here context hook can be used to interact with created context.

```
import React, { useContext } from 'react';
const TestContext = React.createContext();

function Display() {
   const value = useContext(TestContext);
   return <div>{value}, I am learning react hooks.</div>;
}

function App() {
   return (
      <TestContext.Provider value={"Kaleem"}>
         <Display />
      </TestContext.Provider>
   );
}
```

Here provided `Context` is accessed by child functional component using **`useContext`**. Here `Context` object is passed for context hook, and its `value` will be accessible in functional component regardless of depth from `Context Provider`.

<hr/>

### Ref Hook

Reference hook can be used to refer React element created by render method. When there are DOM changes, ref’s .current value will be up to date.

```
import React, { useRef } from 'react';

function App() {
  const inputElement = useRef(null);
  const onButtonClick = () => {
    inputElement.current.focus();
  };
  return (
    <>
      <input ref={inputElement} type="text" />
      <button onClick={onButtonClick}>Focus to input</button>
    </>
  );
}
```

<hr/>

### Reducer Hook

This can act as an **alternative for state hook**. All state changes are bundled in to central function called **reducer**, and state will be updated **according to initiated action and existing state**. Developers coming from Redux, this approach should feel familiar and state hook example is rewritten as this:

```
import React, { useReducer } from 'react';
const initialState = {count: 0};

function reducer(state, action) {
  switch (action.type) {
    case 'inc': return { count: state.count + 1 };
    case 'decs': return { count: state.count - 1 };
    default: throw new Error();
  }
}

function App() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      <h2>Count: {state.count}</h2>
      <button onClick={() => dispatch({type: 'inc'})}>+</button>
      <button onClick={() => dispatch({type: 'decs'})}>-</button>
    </>
  );
}
```

<hr/>

Lets sum up major hooks introduced here, and their usage in making stateful smart functional components.

```
- Basic Hooks -

State Hook → useState()
Add state to functional components and preserve during re-renders.

Effect Hook → useEffect()
Functionality to perform side effects in each renders.

Context Hook → useContext()
Accepts a React.createContext object and returns its current value.

_____________________________________________________
useRef . useReducer . useCallback . many more hooks...

React Hooks API: https://reactjs.org/docs/hooks-reference.html
Create custom hook: https://reactjs.org/docs/hooks-custom.html
```

<br/><br/>
