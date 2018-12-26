# React/Redux Guidelines and Conventions

Refer to the following repositories for reference implementation:

- [React Starter with Material Design](https://github.com/emiketic/emiketic-starter-react/)
- [React Starter with Ant Design](https://github.com/emiketic/emiketic-starter-react-antd/)
- [React Native Starter](https://github.com/emiketic/emiketic-starter-react-native/)

and check `Auth` and `Home` modules for concrete example.

## Repository Structure

Organise repository **by module** (feature set, topic, ...):

- All state management, components, and screens should be fragmented **by module** and placed within dedicated module folder.
- Shared state management and components should be placed under `store` module.

```sh
src/
├── index.css
├── index.js
├── App.js
├── App.css
├── common
│   ├── config.js
│   └── ...
├── store
│   ├── index.js
│   └── state.js
└── Home
    ├── index.js
    ├── state.js
    └── HomeView.js
└── Shared
    ├── index.js
    ├── state.js
    ├── Activity.state.js
    └── ...
```

## JavaScipt/React

- Avoid default export unless advised otherwise.

## State Management with Redux

- State management should be fragmented **by module** and composed in `store/state.js`.

- State name should match module name.

- _Action types_, _action creators_, and _reducer_ should be defined within the **same file** named `state.js`.

- _Action types_ must be prefixed with module name.

- State management definition should export **only** _**exposed** action creators_ (public interface), _reducer_ , _persister_, and event handlers.

- Asynchronous operation' success and failure side effects (alerts, toasts, ...) must be handled in view layer

A typical asynchronous operation must be defined using [`redux-thunk`](https://github.com/gaearon/redux-thunk) as follows:

- a request _action type_ (`MODULE_OPERATION_REQUEST`)
- a request _**internal** action creator_ (`operationRequest()`)
- a success _action type_ (`MODULE_OPERATION_SUCCESS`)
- a success _**internal** action creator_ (`operationSuccess({ ... })`)
  - returns a value - or a promise with a value - to be passed to view layer by _**exposed** action creator_
- a failure _action type_ (`MODULE_OPERATION_FAILURE`)
- a failure _**internal** action creator_ (`operationFailure(error)`)
  - throws an error to be passed to view layer by _**exposed** action creator_
- an _**exposed** action creator_ (`$operation(param1, param2, ...)`)
  - executes the asynchronous operation
  - returns a promise that communicates progress and outcome using _**internal** action creators_
  - name should start with `$`
- a handling for request, success and failure _action types_ within the reducer

**Example:** `Home/state.js`

```javascript
import * as FetchHelper from '../common/fetch.helper';
import * as Activity from '../common/Activity.state';

/**
 * Module Name
 */

export const MODULE = 'Home';

/**
 * Initial State
 */

const INITIAL_STATE = {
  tasks: null,
};

/**
 * Fetch Tasks
 */

const HOME_TASK_FETCH_INDEX_REQUEST = 'HOME_TASK_FETCH_INDEX_REQUEST';
const fetchTaskIndexRequest = (params) => ({ type: HOME_TASK_FETCH_INDEX_REQUEST, ...params });

const HOME_TASK_FETCH_INDEX_SUCCESS = 'HOME_TASK_FETCH_INDEX_SUCCESS';
const fetchTaskIndexSuccess = (data) => ({ type: HOME_TASK_FETCH_INDEX_SUCCESS, data });

const HOME_TASK_FETCH_INDEX_FAILURE = 'HOME_TASK_FETCH_INDEX_FAILURE';
const fetchTaskIndexFailure = (error) => ({ type: HOME_TASK_FETCH_INDEX_FAILURE });

export function $fetchTaskIndex() {
  return (dispatch) => {
    dispatch(Activity.$processing());
    dispatch(fetchTaskIndexRequest());

    return fetch('https://httpbin.org/ip')
      .then(FetchHelper.processResponse, FetchHelper.processError)
      .then((result) => dispatch(fetchTaskIndexSuccess({ tasks: result })))
      .catch((error) => dispatch(fetchTaskIndexFailure(error)))
      .finally(() => dispatch(Activity.$done()));
  };
}

/**
 * Reducer
 */

export function reducer(state = INITIAL_STATE, action) {
  switch (action.type) {
    case HOME_TASK_FETCH_INDEX_REQUEST:
      return {
        ...state,
        tasks: null,
      };
    case HOME_TASK_FETCH_INDEX_SUCCESS:
      return {
        ...state,
        tasks: action.data,
      };
    case HOME_TASK_FETCH_INDEX_FAILURE:
      return {
        ...state,
        tasks: null,
      };
    default:
      return state;
  }
}

/**
 * Persister
 */

export function persister({ tasks }) {
  return {
    tasks,
  };
}
```

## Components

- Prefer stateless functional components (UI as pure function of the props).
- Rely on `react-redux`' `connect()` to attach state and actions to components props.
  - `connect()` should only expose state using `mapStateToProps` and do not define `mapDispatchToProps`, unless the component is defined with a function.
- Rely on `react-router`' `withRouter()` HOC to get routing props (`match`, `location`, `history`).

### Redux-connected Components

- functions defined in `mapDispatchToProps` should be used directly in render function

### View Components

- Export as _default export_.
- Rely on `react-router` props (`match`, `location`, `history`), provided by `<Route />`, for navigation.

**Example:** `Home/HomeView.js`

```javascript
import React, { Component } from 'react';
import { connect } from 'react-redux';

import * as Activity from '../common/Activity.state';

import { $fetchTaskIndex } from './state';

const withStore = connect((state) => ({
  tasks: state.Home.tasks,
}));

const Wrapper = (C) => withStore(C);

class HomeView extends Component {
  componentDidMount() {
    this.load();
  }

  load() {
    const { dispatch } = this.props;
    dispatch($fetchTaskIndex())
      .then(() => dispatch(Activity.$toast('success', 'Tasks loaded')))
      .catch((error) => dispatch(Activity.$toast('failure', error.message)));
  }

  render() {
    return <pre>{JSON.stringify(this.props.tasks, null, 2)}</pre>;
  }
}

export default Wrapper(HomeView);
```

### Non-View Components

...

#### Form Components

...

#### Modal Components

...

## Libraries

- [`react-router`](https://reacttraining.com/react-router/)
- [`react-redux`](https://github.com/reactjs/react-redux)
- [`redux-thunk`](https://github.com/gaearon/redux-thunk)
