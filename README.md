# Style Guide

## Suggested Reading
- [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript)
- [Storybook](https://storybook.js.org/)
- [React-Redux](https://react-redux.js.org/)

## VS Code Extensions
[eslint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
Integrates ESLint into VS Code.

[prettier](https://prettier.io/) Prettier is an opinionated code formatter. It enforces a consistent style by parsing your code and re-printing it with its own rules that take the maximum line length into account, wrapping code when necessary.

[multi-cursor-case-preserve](https://marketplace.visualstudio.com/items?itemName=Cardinal90.multi-cursor-case-preserve)
Have you ever tried to change a single word in all variable names, but had your camelCase broken? This extension preserves selection case in these situations. It recognises CAPS, Uppercase and lowercase. Works for typing or pasting.

[tab out](https://marketplace.visualstudio.com/items?itemName=albert.TabOut)
Tab out of quotes, brackets, etc for Visual Studio Code.

[template-string-converter](https://marketplace.visualstudio.com/items?itemName=meganrogge.template-string-converter)
Converts a string to a template string when ${ is typed


# Code Examples

## Constants {#constants}
Path: src/constants/[state name].constants.js

export const [name] = [name of app | app]/[state name]/[name]
```
export const FETCH_RECORD = 'app/records/FETCH_RECORD';
```

## Action Creators and Thunks
Path: src/actions/[state name].actions.js

The thunks and action creators are usually combined in the same location because the thunk will often be dispatching the actions. 

Things to note
 - The number arguments passed to the actions should be limited
 - The action has a payload which is traditionally a single value to reduce complexity in the reducer.
 - Pagination actions should accept two objects, pagination data and options
   - The incoming pagination data will be reduced from the current state and defaults
   - The options are to be used by the thunk for additional functionality


#### Examples

Single Record Request
```
/**
 * Action Creator for fetching a single record
 * @param {String} id The id of the record to be selected
 * @returns Object
 */
export const fetcRecordAction = (id) => ({
  type: FETCH_RECORD,
  payload: id,
});

/**
 * Action Creator for the success of fetching a single record
 * @param {Promise} response An http respons from fetch
 * @returns Object
 */
export const fetcRecordSuccessAction = (response) => ({
  type: FETCH_RECORD_SUCCESS,
  payload: response,
});

/**
 * Action Creator for the failure of fetching a single record
 * @param {HttpError} error An http respons from fetch
 * @returns Object
 */
export const fetcRecordFailureAction = (error) => ({
  type: FETCH_RECORD,
  payload: error,
});

export const fetchRecord = (id) => async (dispatch) => {
  try {
    // Set the loading state, etc..
    dispatch(fetchRecrodAction());

    // Get the t function from i18n for the liens namespace
    const t = i18n.getFixedT(null, 'liens');

    // Call the service to duplicate the title
    const { resultInfo, ...restResponse } = await fetchRecrodService(id);

    // If we get a 200, dispatch a successfull action and return
    if (resultInfo?.httpStatusCode === 200) {
      dispatch(fetchRecrodSuccessAction(restResponse));
      return true;
    }

    // If the request was unsuccessful, format an error message and throw it.
    throw new ResultInfoError(resultInfo.errors);
  } catch (e) {
    dispatch(enableSnackBarAlert([e.name, e.message].filter((val) => val).join(' - '), 'error'));
    dispatch(fetchLienFiledFailureAction(e));
  }
  return false;
};
```

#### Example Pagination Action

```
export const fetchRecordsAction = (paginationData) => ({
  type: FETCH_RECORDS,
  payload: paginationData,
});

export const fetchRecordsSuccessAction = (response) => ({
  type: FETCH_RECORDS_SUCCESS,
  payload: response,
});

export const fetchRecordsFailureAction = (error) => ({
  type: FETCH_RECORDS_FAILURE,
  payload: error,
});

/**
 * Action Thunk for fetching multiple records
 * @param {Object} paginationData An object of pagination data
 * @param {Object} options An object of additional options for the thunk
 * @returns Boolean
 */
export const fetchRecords = (paginationData = {}, options = {}) => async (dispatch, getState) => { // eslint-disable-line
  try {
    // Set the loading state, etc..
    dispatch(fetchRecordsAction(paginationData));

    // Get the t function from i18n for the titleDetails namespace
    const t = i18n.getFixedT(null, 'titleDetails');

    // Call the service to duplicate the title
    const {
      resultInfo,
      ...restResponse
    } = await fetchRecordsService(coalecedPaginationData);

    // If we get a 200, dispatch a successfull action and return
    if (resultInfo?.httpStatusCode === 200) {
      dispatch(fetchRecordsSuccessAction(restResponse.lenderTitlePage));
      return true;
    }

    // If the request was unsuccessful, format an error message and throw it.
    throw new ResultInfoError(resultInfo.errors);
  } catch (e) {
    dispatch(enableSnackBarAlert(e.message, 'error'));
    dispatch(fetchRecordsFailureAction(e));
  }
  return false;
};
```

## Reducers {#reducers}
#### Path: src/reducers/[state name].reducer.js
#### Snippets: fm_reducer_*
#### Plop: `yarn plop reducer`

Example Reducer for Pagination Data
```
const initialState = {
  records: {
    content: [],
    loading: false,
    error: null,
    searchParams: {},
    pageable: {
      sort: {
        sorted: false,
        unsorted: true,
        empty: true,
      },
      offset: 0,
      pageNumber: 0,
      pageSize: 20,
      paged: true,
      unpaged: false,
    },
    totalElements: 0,
  },
  currentRecord: {
    loading: false,
    error: null,
    record: null,
  }
}

const RecordsReducer = (state = initialState, action) => {
  const { payload, type } = action;
  switch (type) {
    case FETCH_RECORDS:
      return {
        ...state,
        records: {
          loading: true,
          error: initialState.liensToRelease.error,
          ...(Object.keys(payload)?.length ? { ...payload } : {}),
        }
      };
    case FETCH_RECORDS_SUCCESS:
      return {
        ...state,
        records: {
          loading: false,
          error: null,
          ...payload,
        }
      };
    case FETCH_RECORDS_FAILURE:
      return {
        ...state,
        records: {
          ...initialState,
          error: payload, 
        }
      };
    case RESET_FETCH_RECORDS:
      return {
        ...state,
        records: initialState,
      }
    default:
      return state;
  }
}
```

## Selectors {#selectors}
#### Path: src/selectors/[state name].selector.js
#### Snippets: fm_selector_*
#### Plop: `yarn plop selector`

Example Single Record State Selector
```
import { createSelector } from 'reselect';

// Get the records key from state that was created in combineReducers
export const getRecordsState = (state) => state?.records;

//Get the top level domains from the records state
export const getRecordsDomain = createSelector(
  getRecordsState,
  ({records}) => records
)
export const getCurrentRecordDomain = createSelector(
  getRecordsState,
  ({currentRecord}) => currentRecord
)

// Get the individual records from state
export const getRecorsContent = createSelector(
  getRecordsDomain,
  ({ content }) => content
)

//Notice the repeat. This gets state.record.currentRecord.record
export const getCurrentRecordRecord = createSelector(
  getCurrentRecordDomain,
  ({record}) => record
)
```

## Services {#services}
Services are async functions that fetch data from the api. 
 - They should either return a Promise or an Error.
 - There should be no logic other than formatting request data in the function
   - All other logic should be done in the action(thunk/saga)
 - If the response.ok is not true, then it's assumed to be an http status code between 400 and 599

Example Single Record Fetch Service
```
export const fetchRecord = async (id) => {
  const api = `/record/${id}`;
  const options = { method: 'GET' };
  const response = await fetchWrapper(api, options);
  if (response.ok) {
    return response.json();
  }
  throw new HttpError(response);
};
```

Example Pagination Fetch Service
```
/**
 * Service funcion to fetch paginated records
 * @param {Object} paginationData An object of pagination data
 * @returns Promise
 */
export const fetchRecords = async (paginationData = {}) => {
  // returns a URL Encoded a pagination object
  
  const api = `/records?${urlEncodedParams}`;
  const options = { method: 'GET' };
  const response = await fetchWrapper(api, options);
  if (response.ok) {
    return response.json();
  }
  throw new HttpError(response);
};
```

Example POST
```
export const saveRecord = async (id, data) => {
  const api = `/records/${id}`;
  const options = {
    method: 'POST',
    ...(data ? { body: JSON.stringify(data) } : {}),
  };
  const response = await fetchWrapper(api, null, options);
  if (response.ok) {
    return response.json();
  }
  throw new HttpError(response);
};
```

## Components {#components}
Plop: `yarn plop component`

#### Structure
- src
  - components
    - ComponentName
      - ComponentName.jsx
      - ComponentName.styles.js
      - ComponentName.stories.jsx
      - ComponentName.test.jsx
      - index.jsx

#### index.js
Snippet: fm_component_index

Example
```
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { useHistory } from 'react-router-dom';

// Actions
import { fetchRecord } from '../../actions/records.actions'

// Selectors
import { getRecord } from '../../selectors/records.selectors';

// Component
import ComponentName from './ComponentName';

export default (props) => {
  const dispatch = useDispatch();
  return React.createElement(ComponentName, {
    ...props,
    history: useHistory(),
    record: useSelector(getRecord()),
    fetchRecord: (id) => dispatch(fetchRecord(id))
  });
};
```

#### ComponentName.jsx
Snippet: fm_component
Example 
```
import { useEffect } from 'react';
import Grid from '@material-ui/core/Grid';
import Typography from '@material-ui/core/Typography';

import useStyles from './ComponentName.styles';

import { useTranslation } from 'react-i18next';

const ComponentName = () => { 
  const classes = useStyles();
  const { t } = useTranslation('componentName');

  useEffect(() => {
    console.log('ComponentName Loaded');
  }, []);

  return (
    <Grid container className={classes.root} data-testid='component-name'>
      <Grid item xs={12}>
        <Typography variant="h4"></Typography>
      </Grid>
    </Grid>
  )
}
ComponentName.propTypes = {
};

ComponentName.defaultProps = {
};
export default ComponentName;
```

#### ComponentName.styles.jsx
Example
```
import { makeStyles } from '@material-ui/styles';

const useStyles = makeStyles(() => ({
  root: {},
}));

export default useStyles;
```


## Storybook
`yarn storybook`
#### Path: src/components/[name]/[name].stories.jsx

Example Storybook 
```
import Component from './ExampleComponent';

export const ExampleComponent = (args) => <Component {...args} />;

ExampleComponent.args = {
  error: null, // Example error
  fetchRecord: (id) => alert(`'Fetching ${id}`), //
  record: null,
  loading: false,
  id: null,
};

ExampleComponent.argTypes = {
  error: {
    options: ['null', 'string', 'Error'],
    mapping: {
      null: null,
      string: 'Something went wrong',
      Error: new Error('Error class'),
    },
    control: { type: 'radio' },
  },
  loading: {
    options: [true, false],
    control: { type: 'radio' },
  },
  titleNumber: {
    type: 'string',
  },
  record: {
    options: ['null', 'record'],
    mapping: {
      null: null,
      record: {
        id: 123,
        name: "Example Record",
      },
    },
    control: { type: 'radio' },
  },
};

export default { title: 'ExampleComponent' };
```