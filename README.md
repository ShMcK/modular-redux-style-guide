# Modular Redux StyleGuide

Conventions that allow you to create **modular Redux**.

Can Redux be scalable? I believe so. Actions, reducers & components can all be
composed in


### Store

1. dynamically load the middleware you want

```js
import { applyMiddleware, createStore } from 'redux';
import reducer from '../reducers';
import thunk from 'redux-thunk';
import createLogger from 'redux-logger';

const middlewares = [thunk]; // add middleware here

const devMode = false; // turn logger on/off
if (devMode) {
  const logger = createLogger();
  middlewares.push(logger);
}

const store: Redux.Store = createStore(
  reducer,
  applyMiddleware(...middlewares)
);

export default store;
```

1. Avoid importing 'store'. Additional imports break the ability to easily create reusable modules.

```js
// BAD
import store from '../store';
```


### Actions

1. **actions** using [Flux Standard Action](https://github.com/acdlite/flux-standard-action)

    `{type: 'NAME', payload: {}, error, status }`

1. *Avoid importing state*. Use **thunks** instead.

```js
// BAD
import store from '../store';
import {PAGE_SET} from './types';

export function pageSet() {
    const {pagePosition} = store.getState();
    dispatch({
      type: TEST_RUN, payload: { pagePosition }
    });
  };
}
```

```js
// GOOD
import {PAGE_SET} from './types';

export function pageSet() {
  return (dispatch, getState) => {
    const {pagePosition} = getState();
    dispatch({
      type: PAGE_SET, payload: { pagePosition }
    });
  };
}
```


### Action Types

1. Name actions by feature (*'NOUN_VERB'*)

  `export const PAGE_SET = 'PAGE_SET';`



### Reducers

1. Name reducers with a noun, using the data they will change

1. Default params

```js
const _page: CR.Page = {
  title: '',
  description: '',
  completed: false,
};

export default function pageReducer(page = _page, action){
  /* ... */
}
```

1.

```js
import {PAGE_SET} from './types';

export default function pageReducer(page = _page, action){
switch (action.type) {

    case PAGE_SET:
      // loaded from action
      const {pagePosition, tutorial} = action.payload;
      // content
      const {title, description, onPageComplete, completed} = tutorial.pages[pagePosition];
      // return a new object
      return {
        title,
        description,
        onPageComplete,
        completed: completed || false
      };

    // default case returns default page
    default:
      return page;
  }
}
```

1. Avoid mutations. Write pure functions.

  - Avoid array mutation with `.concat` over `.push`

```js
  var array = [1, 2, 3];

  // bad: mutates array
  array.push(4);     // 4
  console.log(array) // [1, 2, 3, 4]

  // good: creates new array
  array.concat(5); // [1, 2, 3, 4, 5]
```

  - Avoid object mutation with `Object.assign`

```js
  var object = { a: '1'};

  // bad: mutates object
  object.b = '2';

  // good: creates new object
  var newObject = Object.assign({}, object, {c: '3'});
```


### Components

Understand smart and dumb components.


### Modules

```js
// pageModule
import * as actions from './actions';
import * as components from './components';
import reducers from './reducers';

export {actions, components, reducers};
```

##### Modular Reducers

```js
// Module
import alert from './alert';
import page from './page';

const reducers = {
   alert, page
};
export default reducers;
```

```js
// Loading reducers
import {reducers} from 'pageModule';
import checks from './checks';

export default combineReducers(
  Object.assign(
    {},
    {checks},
    reducers
  )
);
```

##### Modular Components

```js
// module
export {default as Page} from './Page';
```

```js
// loading components
import {components} from 'pageModule';

export const Page = components.Page;
```
