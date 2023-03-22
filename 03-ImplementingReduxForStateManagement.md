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

## Product List Reducer & Action
- Create the `reducers` folder in `src`
- Create `productReducers.js` in the `reducers` directory and fill as below...

```
export const producListReducer = (state = { products:[] }, action) => {
    switch(action.type){
        case 'PRODUCT_LIST_REQUEST':
            return {loading: true, products:[] }
            
        case 'PRODUCT_LIST_SUCCESS':
            return {loading: false, products: action.payload }
            
        case 'PRODUCT_LIST_FAIL':
            return {loading: false, error: action.payload }
            
        default:
            return state
    }
}
```

- Go back to `store.js` and add as below...

```
...
import { productListReducer } from './reducers/productReducers'

const reducer = combineReducers({
    productList: productListReducer,
})
```

- Check redux devtools again and you should be able to see the empty products array.
