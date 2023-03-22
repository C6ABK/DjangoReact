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
        case PRODUCT_LIST_REQUEST:
            return {loading: true, products:[] }
            
        case PRODUCT_LIST_SUCCESS:
            return {loading: false, products: action.payload }
            
        case PRODUCT_LIST_FAIL:
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

### Constants
- Create `constants` folder in `src`
- Create `productConstants.js` in `src/constants`, fill as below

```
export const PRODUCT_LIST_REQUEST = 'PRODUCT_LIST_REQUEST'
export const PRODUCT_LIST_SUCCESS= 'PRODUCT_LIST_SUCCESS'
export const PRODUCT_LIST_FAIL = 'PRODUCT_LIST_FAIL'
```

- Import into `productReducers.js` as below...

```
import axios from 'axios'
import {
    PRODUCT_LIST_REQUEST,
    PRODUCT_LIST_SUCCESS,
    PRODUCT_LIST_FAIL
} from '../constants/productConstants'
```

### Actions
- Create `actions` folder in `src`
- Create `productActions.js` and fill as below
- <b>NOTE</b> - USE FULL PATH BELOW FOR CODEANYWHERE WHEN MAKING API CALL 

```
import {
    PRODUCT_LIST_REQUEST,
    PRODUCT_LIST_SUCCESS,
    PRODUCT_LIST_FAIL
} from '../constants/productConstants'

export const listProduct = () => async (dispatch) => {
    try{
        dispatch({ type: PRODUCT_LIST_REQUEST })
        
        const { data } = await axios.get('/api/products/')
        
        dispatch({
            type: PRODUCT_LIST_SUCCESS,
            payload: data
        }
    } catch(error){
        dispatch({ 
            type: PRODUCT_LIST_FAIL,
            payload: error.response && error.response.data.message 
                ? error.response.data.message 
                : error.message
        })
    }
```

## Bringing Redux into Home Screen
- Open `HomeScreen.js` and delete the axios import, products state and everything inside useEffect
- Modify as below...

```
...
import  { useDispatch, useSelector } from 'react-redux'
import { listProducts } from '../actions/productActions'
...

function HomeScreen() {
    const dispatch = useDispatch()
    const productList = useSelector(state => state.productList)
    const { error, loading, products } = productList
    
    useEffect(() => {
        dispatch(listProducts())
    }, [])
}

```
