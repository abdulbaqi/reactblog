This app will fetch some fake API data from
http://jsonplaceholder.typicode.com/
and use two endpoints `posts` and `users`.

After creating the app we need to add the following four dependencies:

```
yarn add redux react-redux axios redux-thunk
```

The `redux-thunk` helps us in making request in the react-redux environment.

As always, we will start with our boiler plate in `src/index.js`

```javascript
import React from "react";
import ReactDOM from "react-dom";
import { Provider } from "react-redux";
import { createStore } from "redux";

import App from "./components/App";
import reducers from "./reducers";

ReactDOM.render(
  <Provider store={createStore(reducers)}>
    <App />
  </Provider>,
  document.querySelector("#root")
);
```

and then in the `src/components/App.js`

```javascript
import React from "react";

const App = () => {
  return <div className="ui container">App</div>;
};

export default App;
```

and finally in the `src/reducers/index.js`

```javascript
import { combineReducers } from "redux";

export default combineReducers({
  replaceMe: () => "i am dummy"
});
```

The above setup is the minimum boiler plate that makes our app work if we use `yarn start`.

## Post List

Now I create a post list that component that will fetch data from API. Here are the typical steps:

1. We need to get the list of posts everytime the component is loaded, hence we need to implement the `componentDidMount()` lifecycle method.

2. Inside the mount method we call action creator

3. that action creator uses `axios` to fetch data from API and load it into the `payload` property. In this context the middleware `redux-thunk` comes into picture

4. some reducers then will watch for the loaded data in payload and dispatch it

5. in this way our react components as part of state change will get those new data. This is when `mapStateToProps` comes.

# action fetch post

in `src/actions/index.js` we create:

```javascript
export const fetchPosts = () => {
  return {
    type: "FETCH_POSTS"
  };
};
```

then we turn to our PostList and wire up the action through `connect` as follows:

```javascript
import React from "react";
import { connect } from "react-redux";

import { fetchPosts } from "../actions";

class PostList extends React.Component {
  render() {
    return <div>Post List</div>;
  }
}

export default connect(
  null,
  { fetchPosts }
)(PostList);
```

note that `export default connect(null,{fetchPosts: fetchPosts})(PostList);` is equivalet under ES 2015 to `export default connect(null,{fetchPosts})(PostList);`

Also, the `null` will eventually will be replaced with `mapStateToProps` later.

And now we can use this `fetchPost` action inside our lifecycle method as follows

```javascript
import React from "react";
import { connect } from "react-redux";

import { fetchPosts } from "../actions";

class PostList extends React.Component {
  componentDidMount() {
    this.props.fetchPosts();
  }
  render() {
    return <div>Post List</div>;
  }
}

export default connect(
  null,
  { fetchPosts }
)(PostList);
```
