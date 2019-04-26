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
import React from 'react';

const App = () => {
    return <div className="ui container">App</div>
};

export default App;

```

and finally in the `src/reducers/index.js`

```javascript
import {combineReducers} from 'redux';

export default combineReducers({});
```

The above setup is the minimum boiler plate that makes our app work if we use `yarn start`.