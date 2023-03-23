# Implementing Redux for State Management
## Create Redux Store
- `npm install redux react-redux redux-thunk redux-devtools-extension`
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
export const productListReducer = (state = { products:[] }, action) => {
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

```
import axios from 'axios'
import {
    PRODUCT_LIST_REQUEST,
    PRODUCT_LIST_SUCCESS,
    PRODUCT_LIST_FAIL
} from '../constants/productConstants'

export const listProducts = () => async (dispatch) => {
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
    }, [dispatch])
}

```

### Loading Message
- In `HomeScreen.js` under the Latest Products H1 title, modify as velow...

```
<h1>Latest Products</h1>
{loading ? <h2>Loading...</h2>
    : error  ? <h3>{error}</h3>
    : 
    <Row>
        {products.map(product => (
            <Col key={product._id} sm={12} md={6} lg={4} xl={3}>
                <Product product={product} />
            </Col>
        ))}
    </Row>
}
```

## Message & Loader Component
- Go to `components` and create `Loader.js` and `Message.js`

### Loader.js
- `_rfce` in `Loader.js` and fill as below

```
import { Spinner } from 'react-bootstrap'

function Loader() {
    return (
        <Spinner
            animation='border'
            role='status'
            style={{
                height: '100px',
                width: '100px',
                margin: 'auto',
                display: 'block'
            }}
        >
            <span className='sr-only'>Loading...</span>
        </Spinner>
    )
}

```

- Go to `HomeScreen.js` and import the loader - `import Loader from '../components/Loader' `
- Replace the original loading message with the `Loader` component...

```
...
<h1>Latest Products</h1>
{loading ? <Loader />
    : error  ? <h3>{error}</h3>
...
```

### Message.js
- `_rfce` to populate and modify as below...

```
import { Alert } from 'react-bootstrap'

function Message({variant, children}) {
    return (
        <Alert variant={variant}>
            {children}
        </Alert>
    )
}
```

- Go back to `HomeScreen.js` and import the message component - `import Message from '../components/Message`
- Wrap the `{error}` in the new `Message` component as below...

```
...
<h1>Latest Products</h1>
{loading ? <Loader />
    : error  ? <Message variant='danger'>{error}</Message>
...
```

## Product Details Reducer & Action
- In the `productConstants.js`, add the following below your existing constants...

```
...
export const PRODUCT_DETAILS_REQUEST = 'PRODUCT_DETAILS_REQUEST'
export const PRODUCT_DETAILS_SUCCESS = 'PRODUCT_DETAILS_SUCCESS'
export const PRODUCT_DETAILS_FAIL = 'PRODUCT_DETAILS_FAIL'

```

- Go to `productReducers.js` and import the new constants...

```
import {
    ...
    PRODUCT_DETAILS_REQUEST,
    PRODUCT_DETAILS_SUCCESS,
    PRODUCT_DETAILS_FAIL,
} from '../constants/productConstants'
```

- Create a new reducer for product details as below...
- NOTE: using `...state` in the `productDetailsReducer` seems to make a previously selected product visible for a short time if you go back and select another. Use `product:[]` to behave the same as the `productListReducer`

```
export const productDetailsReducer = (state = { product: {reviews:[]} }, action) => {
    switch (action.type) {
        case PRODUCT_DETAILS_REQUEST:
            return { loading: true, ...state }
        
        case PRODUCT_DETAILS_SUCCESS:
            return { loading: false, product: action.payload }
            
        case PRODUCT_DETAILS_FAIL:
            return { loading: false, error: action.payload }
            
        default:
            return state
    }
}

```

- Open `store.js` and modify as below...

```
...
import { productListReducer, productDetailReducer } from './reducers/productReducers'

const reducer = combineReducers({
    productList: productListReducer,
    productDetails: productDetailsReducer,
})
...

```

- Open `productActions.js` and modify as below...

```
...
import {
    PRODUCT_LIST_REQUEST,
    PRODUCT_LIST_SUCCESS,
    PRODUCT_LIST_FAIL,
    
    PRODUCT_DETAILS_REQUEST,
    PRODUCT_DETAILS_SUCCESS,
    PRODUCT_DETAILS_FAIL,
} from '../constant/productConstants'

export const listProducts = () => async (dispatch) => {
    ...
}

export const listProductDetails = (id) => async (dispatch) => {
    try {
        dispatch({ type: PRODUCT_DETAILS_REQUEST })
        
        const { data } = await axios.get(`/api/products/${id}`)
        
        dispatch({
            type: PRODUCT_DETAILS_SUCCESS,
            payload: data
        })
    } catch (error) {
        dispatch({
            type: PRODUCT_DETAILS_FAIL,
            payload: error.response && error.response.data.message 
                ? error.response.data.message 
                : error.message
        })
    }
}

```

- Go to `ProductScreen.js`
- Get rid of the `axios` import and `product` state and modify as below...
- REPLACE MATCH WITH PRODUCT_ID

```
...
import { useDispatch, useSelector } from 'react-redux'
import { listProductDetails } from '../actions/productActions'
import Loader from '../components/Loader'
import Message from '../components/Message

function ProductScreen({ match }) {
    const dispatch = useDispatch()
    const productDetails = useSelector(state => state.productDetails)
    const { loading, error, product } = productDetails
    
    useEffect(() => {
        dispatch(listProductDetails(match.params.id)
    }, [dispatch, match])
}

return (
    <div>
        <Link to='/' className='btn btn-light my-3'>Go Back</Link>
        
        {loading ?
            <Loader />
            : error
                ? <Message variant='danger'>{error}</Message>
            : (
                <Row>
                    ...
                </Row>
            )
        }
    </div>
)

```
