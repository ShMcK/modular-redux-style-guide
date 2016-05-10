# Modular Redux StyleGuide

Conventions that allow you to create **modular Redux**. A Redux application may even broken up into smaller NPM modules.

Can Redux be scalable? I believe so. **Actions**, **reducers** & **components** can all be composed into smaller, feature based modules with their own isolated data trees.

For React/Redux examples, see the [Atom-CodeRoad](https://github.com/coderoad/atom-coderoad) codebase.

## Index

1. [Store](#store)
1. [Actions](#actions)
1. [Action Types](#action-types)
1. [Reducers](#reducers)
1. [Components](#components)
1. [Modules](#modules)


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

2. Avoid importing 'store'. Additional imports break the ability to easily create reusable modules.

```js
// BAD
import store from '../store';
```


### Actions

1. **actions** using [Flux Standard Action](https://github.com/acdlite/flux-standard-action)

    `{type: 'NAME', payload: {}, error, status }`

    - **type**: the action name
    - **payload**: all necessary variables to pass to reducers
    - **error**: error handling. May be a boolean or an object
    - **status**: often used with [Redux Promise](https://github.com/acdlite/redux-promise) (pending, resolved, etc.)

2. *Avoid importing state*. Use **thunks** instead.

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

3. Treat action creators as dispatchers, rather than piling logic in multiple reducers across action types.


### Action Types

1. Name actions by feature (*'NOUN_VERB'* using *UPPER_SNAKE_CASE*)

  `export const PAGE_SET = 'PAGE_SET';`



### Reducers

1. Name reducers as *noun + Reducer* ('pageReducer')

2. Use [Default params](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Default_parameters)

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

3. Limit the number of action types within a reducer. Use action dispatchers instead.

4. Use `switch` statements with a `default` case

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

5. Consider using 'filters' to simplify reducers while maintaining readability.

```js
case TEST_RESULT:
      const result = action.payload.result;

      switch (action.payload.filter) {

        case 'PASS':
          return setAlert({
            message: result.msg,
            action: 'pass',
            duration: result.duration || 1200,
          }, '#73C990');

        case 'FAIL':
          return setAlert({
            message: result.msg,
            action: 'fail',
            duration: result.duration || 2200,
          }, '#FF4081');
      }
      // Note
      return setAlert({
        message: result.msg,
        duration: result.duration || 2200,
      }, '#9DA5B4');

    default:
      return alert;
  }
```

5. Avoid mutations. Write **pure functions**. Pure functions are predictable, meaning fewer bugs.

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

React apps are generally broken into smart and dumb components.

- **dumb**: components that do not change data. These do not need to be connected to Redux. Dumb components can generally be created as [stateless functions](https://facebook.github.io/react/docs/reusable-components.html#stateless-functions)

- **smart**: these components can trigger actions. They can be setup using [react-redux](https://github.com/reactjs/react-redux).

Smart components should be as atomic as possible, allowing for greater re-use. This may mean only the smallest tag, such as an event button, connects to the Redux store. Changes are propagated through 'props', and storing data in 'state' should be avoided.

'React-redux' provides an optional decorator syntax (`@connect`).

```js
import * as React from 'react';
import {connect} from 'react-redux';
import {pageNext} from '../actions';

// null triggers the PureRenderMixin
@connect(null, (dispatch, getState) => {
  return {
    callNextPage: () => {
      dispatch(pageNext());
    }
  };
})
export default class Continue extends React.Component {
  render() {
    return (
      <button onClick={this.props.callNextPage}>Continue</button>
    );
  }
}
```

### Modules

Redux applications can be composed in such a way that reducers and actions can be included as modules or even NPM packages.

Modules can include any combination of *actions*, *reducers* and *components*.

See an example of an exported module below:

```js
// pageModule
import * as actions from './actions';
import reducers from './reducers';
import * as components from './components';


export {actions, reducers, components};
```

##### Exporting Actions

Simply export your module actions.

```js
export {pageSet, pageNext, pagePositionSet} from './page';
export {alertToggle, alertReplay} from './alert';
```

##### Modular Reducers

Reducers are merely objects composed of reducer names (keys) and reducers (values). Reducers can be easily exported.

```js
// Module
import alert from './alert';
import page from './page';

const reducers = {
   alert, page
};
export default reducers;
```

Reducer objects can then be imported from a module, and combined into the list of app reducers.

```js
// Loading module reducers
import {reducers} from 'pageModule';
// local reducer
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

Similarly, components can be exported from a module.

```js
// module
export {default as Page} from './Page';
```

Then components may be imported into a local index file, and imported into a component.

```js
// loading components
import {components} from 'pageModule';

export const Page = components.Page;
```
