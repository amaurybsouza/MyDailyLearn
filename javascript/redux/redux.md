# Redux

#### What is Redux

Redux is a state management framework that can be used with a number of different web technologies, including React.

In Redux, there is a `single state object` that's responsible for the `entire` state of your application. This means if we had a React app with ten components, and each component had its own local state, the entire state of your app would be defined by a single state object housed in the Redux `store`.

`Store` is the single source of truth when it comes to application state.

```js
// reducer function takes state as an argument and returns `state`
const reducer = (state = 5) => {
  return state;
}

// create a redux store
const store = Redux.createStore(reducer);
```

#### Get State from the Redux Store

Use `store.getState()` to retrieve the `state` from the `store`.

```js
const store = Redux.createStore((state = 5) => state);

// get state from the store
const currentState = store.getState();
```

#### Redux Action

- An `action creator` is simply a JavaScript function that returns an action. In other words, action creators create objects that represent action events
- `dispatch` method is what you use to dispatch actions to the Redux store

```js
const store = Redux.createStore(
  (state = {login: false}) => state
);

// action creator
const loginAction = () => {
  return {
    type: 'LOGIN'
  }
};

// dispatch the action
store.dispatch(loginAction())
```