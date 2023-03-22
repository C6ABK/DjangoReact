# Implementing Redux for State Management
## Create Redux Store
- `npm install redux react-redux redux-thunk redux-devtools-extensions`
- Go to `frontend/src` and create `store.js` and add the code below...

```
import { createStore, combineReducers, applyMiddleware } from 'redux'
import thunk from 'redux-thunk'
import { composeWithDevTools } from 'redux-devtools-extension'

const reducer = combineReducers({})

const initialState = {}

const middleware = [thunk]

const store = createStore(reducer, initialState, composeWithDevTools(applyMiddleware(...middleware)))

export default store
```

- Go to `index.js` and modify as below...

```
...
import { Provider } from 'react-redux'
import store from './store'
...
ReactDOM.render(
    <Provider store={store}>
        <App />
    </Provider>
    document.getElementById('root')
)
...
```
- Inspect > security > redux and you should be able to see the store.
