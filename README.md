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

## API

create a folder called `api` and inside `jsonplaceholder.js` with the following content:

```javascript
import axios from "axios";

export default axios.create({
  baseURL: "http://jsonplaceholder.typicode.com"
});
```

Then inside the action we hookup the axios and return the payload as follows:

```javascript
import jsonPlaceholder from "../apis/jsonPlaceholder";

export const fetchPosts = async () => {
  const response = await jsonPlaceholder.get("/posts");
  return {
    type: "FETCH_POSTS",
    payload: response
  };
};
```

But this will return an error, because this action `fetchPosts` is not returning a plain JS object. You can prove that by placing the code above in bablejs. This is because of the `async` and `await` construct.

one way to deal with it - and it would remove error message- is to return a `promise` in the payload as follows:

```javascript
import jsonPlaceholder from "../apis/jsonPlaceholder";

export const fetchPosts = () => {
  const promise = jsonPlaceholder.get("/posts");
  return {
    type: "FETCH_POSTS",
    payload: promise
  };
};
```

but the above scenario -although we do not have error- it will not work because this action will be dispatched to reduces without the data is fetched yet from the api.

## redux thunk as a middle ware

now with the `redux-thunk` middleware the dispatch will send the promise to the middleware which will wait for data before sending that to the reducers.

Redux thunk can take an object or a function from the action creator. Only when it gets a function that it starts to care about.

This function access the dispatch and manually invokes it when the data is returned from API.

now here are the steps to get `redux-thunk`. First in the `index.js` in `src/index.js` we import

```javascript
import React from "react";
import ReactDOM from "react-dom";
import { Provider } from "react-redux";
import { createStore, applyMiddleware } from "redux";
import thunk from "redux-thunk";

import App from "./components/App";
import reducers from "./reducers";

const store = createStore(reducers, applyMiddleware(thunk));

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.querySelector("#root")
);
```

and then the `action/index.js` becomes:

```javascript
import jsonPlaceholder from "../apis/jsonPlaceholder";

export const fetchPosts = () => {
  return async function(dispatch, getSate) {
    const response = await jsonPlaceholder.get("/posts");
    dispatch({
      type: "FETCH_POSTS",
      payload: response
    });
  };
};
```

which can further be refactored as:

```javascript
import jsonPlaceholder from "../apis/jsonPlaceholder";

export const fetchPosts = () => async dispatch => {
  const response = await jsonPlaceholder.get("/posts");
  dispatch({
    type: "FETCH_POSTS",
    payload: response
  });
};
```

In this way we have wired-up redux-thunk successfully.

## getting the reducer data off the payload

so let us create a `postReducer.js` file inside the `src/reducer` folder.

The objective of this reducer is to receive the data from middleware dispatcher and load an array that will hold the data about posts.

initailly let us test that it works

```javascript
export default () => {
  return 123;
};
```

and hook it up in the `src/reducers/index.js`

```javascript
import { combineReducers } from "redux";
import postReducer from "./postReducer";

export default combineReducers({
  post: postReducer
});
```

reducer must return anything other than 'undefined'.

also reducer produce state that are results of modifying other prevous state or action.

in other words, a reducer gets a previous state and an action, and does something, and then as output results in a revised state.

a third rule is that a reducer can only consider the new action and the current state to decide what to do, and never get our of itself and call some API, or reach out to DOM or get user input or any thing else. That means keep the reducer pure.

The last rule is that the reducer never mutate the state that it receives, like by push or pop on an array for example. Or doing assignments of objects.

as a side note: in javascript numbers and strings are immutable.

another side note, is the `===` comparision between arrays in js is checking if both are fererencing the same array or object

for example the follwoing is false

```javascript
const numbers = [1, 2, 3];
numbers === [1, 2, 3]; //return false because === does not check the content, just the reference
```

So, here are some ways we can return objects or arrays

```javascript
state.pop() // bad
state.filter(element => element !== 'hi') //good

state.push('hi') //bad
[...state, 'hi'] //good

state[0]='hi' //bad
state.map(el => el==='hi'?'bye':el)//good

state.name = 'sam' //bad
{...state, name:'sam'} //good

delete state.name //bad
{...state, age:undefined}//good
_.omit(state, 'age') //good
```

given the above we have this version of postReducer.js

```javascript
export default (state = [], action) => {
  switch (action.type) {
    case "FETCH_POSTS":
      return action.payload;
    default:
      return state;
  }
};
```

## mapStateToProps

now that we have all the wires in place, here is the new look of PostList.js

```javascript
import React from "react";
import { connect } from "react-redux";

import { fetchPosts } from "../actions";

class PostList extends React.Component {
  componentDidMount() {
    this.props.fetchPosts();
  }
  render() {
    console.log(this.props.posts);
    return <div>post list will load here</div>;
  }
}

const mapStateToProps = state => {
  return { posts: state.posts };
};

export default connect(
  mapStateToProps,
  { fetchPosts }
)(PostList);
```

A small change we need to adjust in our action is to assign the payload with values of response.data just not to overburder with unncessary data.

### helper function to list post in the render

just using the right semantic UI classes as follows

```javascript
renderList() {
    return this.props.posts.map(post => {
      return (
        <div className="item" key={post.id}>
          <i className="large middle aligned icon user" />
          <div className="content">
            <div className="description">
            <h2>{post.title}</h2>
            <p>{post.body}</p>

            </div>
          </div>
        </div>
      );
    });
  }
  render() {
    console.log(this.props.posts);
    return <div className="ui relaxed divided list">{this.renderList()}</div>;
  }
```

## Fetching users

Our design dictates that we need to list the name of author against each post. That means we need to call another API with endpoint `/users/:id`. For that we need first to create anther action creator `fetchUser` and then anther reducer called `usersReducer`. 

so first, we add a new action `FETCH_USER` as follows:

```javascript
import jsonPlaceholder from "../apis/jsonPlaceholder";

export const fetchPosts = () => async dispatch => {
  const response = await jsonPlaceholder.get("/posts");
  dispatch({
    type: "FETCH_POSTS",
    payload: response.data
  });
};

export const fetchUser = id => async dispatch => {
  const response = await jsonPlaceholder.get(`/users/${id}`);
  dispatch({ type: "FETCH_USER", payload: response.data });
};
```

Then, we need to create a new reducer as follows in `src/reducers/usersReducer.js`

```javascript
export default (state = [], action) => {
  switch (action.type) {
    case "FETCH_USER":
      return [...state, action.payload];
    default:
      return state;
  }
};
```

Note how we returned the state by ammending the payload. 

Finally we can create a user header component as follows:

```javascript
import React from "react";
import { connect } from "react-redux";
import { fetchUser } from "../actions";

class UserHeader extends React.Component {
  componentDidMount() {
    this.props.fetchUser(this.props.userId);
  }
  render() {
    const user = this.props.users.find(user => user.id === this.props.userId);
    if (!user){
      return null;
    }
    return <div className="header">{user.name} </div>;
  }
}

const mapStateToProps = state => {
  return { users: state.users };
};

export default connect(
  mapStateToProps,
  { fetchUser }
)(UserHeader);
```
which will be displayed inside our original posts compenents as follows:

```javascript
//truncated code above
renderList() {
    return this.props.posts.map(post => {
      return (
        <div className="item" key={post.id}>
          <i className="large middle aligned icon user" />
          <div className="content">
            <div className="description">
            <h2>{post.title}</h2>
            <p>{post.body}</p>

            </div>
          </div>
        </div>
      );
    });
  }

  //rest of the code
  ```
  We can now do a small refactoring in our userHeader as follows 

  ```javascript
  import React from "react";
import { connect } from "react-redux";
import { fetchUser } from "../actions";

class UserHeader extends React.Component {
  componentDidMount() {
    this.props.fetchUser(this.props.userId);
  }
  render() {
    const {user} = this.props;
    if (!user){
      return null;
    }
    return <div className="header">{user.name} </div>;
  }
}

const mapStateToProps = (state, ownProps) => {
  return { user: state.users.find(user=>user.id === ownProps.userId) };
};

export default connect(
  mapStateToProps,
  { fetchUser }
)(UserHeader);
```

Note how we have a second argument `ownProps`. 

Note also that the above solution will cause the fetch of users with each of 100 blog post although we have only 10 users. So, how to solve this redundency? 

One way is to use `memoize` function of `lodash`. 

When you open chrome console while in lodash website, then the library is loaded automatically for you. Also, when you want a new line press 'shift' and 'enter'.

So, let us install lodash.

```
yarn add lodash
```
## Memoize function

in the fetchUser action `src/actions/index.js` we use the right way to memoize as follows:

```javascript
import _ from "lodash";
import jsonPlaceholder from "../apis/jsonPlaceholder";

export const fetchPosts = () => async dispatch => {
  const response = await jsonPlaceholder.get("/posts");
  dispatch({
    type: "FETCH_POSTS",
    payload: response.data
  });
};

export const fetchUser = id => dispatch => {
  _fetchUser(id, dispatch);
};

const _fetchUser = _.memoize(async (id, dispatch) => {
  const response = await jsonPlaceholder.get(`/users/${id}`);
  dispatch({ type: "FETCH_USER", payload: response.data });
});
```

With this solution, we can not re-fetch a user request again. 

Here is a second way to do this.

### fetchPostAndUser action creator

In this alternative approach we create this new fetchPostAndUsers() which will: 1) call fetchPost, 2)get list of posts 3) fill all unique user IDs from the list of posts 4)iterave of those uniqe ids and 5) call fetch user with each userID

we can use lodash's own map on arrays as well as uniq. So, the following code the `map` grabs all userId, and then passes it to `uniq` to get the unique ids only. 

```javascript
_.uniq(_.map(getState().posts, 'userId'))
```

so here is how to the actions look like

```javascript
import _ from "lodash";
import jsonPlaceholder from "../apis/jsonPlaceholder";

export const fetchPostsAndUsers = () => async (dispatch,getState) => {
  // console.log('about to fetch...')
  await dispatch(fetchPosts());
  const userIds = _.uniq(_.map(getState().posts, 'userId'));
  // console.log(userIds);
  userIds.forEach(id => dispatch(fetchUser(id)));
};

export const fetchPosts = () => async dispatch => {
  const response = await jsonPlaceholder.get("/posts");
  dispatch({
    type: "FETCH_POSTS",
    payload: response.data
  });
};


export const fetchUser = id => async dispatch => {
  const response = await jsonPlaceholder.get(`/users/${id}`);
  dispatch({ type: "FETCH_USER", payload: response.data });
};

```

So, now we can remove the call to fetchuser from  userHeader

```javascript
import React from "react";
import { connect } from "react-redux";
// import { fetchUser } from "../actions";

class UserHeader extends React.Component {
  // componentDidMount() {
  //   this.props.fetchUser(this.props.userId);
  // }
  render() {
    const {user} = this.props;
    if (!user){
      return null;
    }
    return <div className="header">{user.name} </div>;
  }
}

const mapStateToProps = (state, ownProps) => {
  return { user: state.users.find(user=>user.id === ownProps.userId) };
};

export default connect(
  mapStateToProps,
  // { fetchUser }
)(UserHeader);

```