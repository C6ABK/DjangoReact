# Adding to Cart
## Qty Select
- Open `ProductScreen.js` and modify as below...
  - Edit the `match` section in this document later

```
...
import { useEffect, useState } from 'react'
import { Link, useParams, useNavigate } from 'react-router-dom' 
import { Row, Col, Image, ListGroup, Button, Card, Form } from 'react-bootstrap'
...

function ProductScreen() {
  const [qty, setQty] = useState(1)
  let navigate = useNavigate()
 
  ...
}
```

- Under the status listgroup item, add the following...

```
...

{product.countInStock > 0 && (
  <ListGroup.Item>
    <Row>
      <Col>Qty</Col>
      <Col xs="auto" className="my-1">
        <Form.Control
          as="select"
          value={qty}
          onChange={(e) => setQty(e.target.value)}
        >
          {
            [...Array(product.countInStock).keys()].map((x) => (
              <option key={x + 1} value={x + 1}>
                { x + 1 } 
              </option>
            ))
          }
        </Form.Control>
      </Col>
    </Row>
  </ListGroup.Item>
)}

...
```

## Add to Cart
- Modify the "Add to Cart" button as below

```
<ListGroup.Item>
  <Button
    onClick={addToCartHandler}
    ...
  >
    ...
  </Button>
</ListGroup.Item>
```

- Next, create the `addToCartHandler` underneath your `useEffect`...

```
...
useEffect(() => {
  ...
}

const addToCartHandler = () => {
  navigate(`/cart/${product_id.id}?qty=${qty}`)
}
...
```

- Add `CartScreen.js` in the `screens` directory and fill with `rfce`
- Import `CartScreen` in `App.js`
- Create a route for `CartScreen` - `<Route path='/cart/:id?' element={<CartScreen />} />`

## Cart Reducer & Action
- Add `cartConstants.js` to `constants`
- Fill as below

```
export const CART_ADD_ITEM = 'CART_ADD_ITEM'
export const CART_REMOVE_ITEM = 'CART_REMOVE_ITEM'
```

- Add `cartReducers.js` to `reducers`
- Fill as below - this checks if the product is already in the cart

```
import { CART_ADD_ITEM } from '../constants/cartConstants

export const cartReducer = (state={cartItems:[]}, action) => {
  switch(action.type){
    case CART_ADD_ITEM:
      const item = action.payload
      const existItem = state.cartItems.find(x => x.product === item.product)
      
      if(existItem){
        return {
          ...state,
          cartItems: state.cartItems.map(x => 
            x.product === existItem.product ? item : x
          )
        }
      } else {
        return {
          ...state,
          cartItems:[...state.cartItems, item]
        }
      }
      
    default:
      return state
  }
}
```
- Go to `store.js` and modify as below...

```
...
import { cartReducer } from './reducers/cartReducers'

const reducer = combineReducers({
  ...
  cart: cartReducer,
})
```

### Actions
- Add `cartActions.js` in the `actions` folder
- Fill as below

```
import axios from 'axios'
import { CART_ADD_ITEM } from '../constants/cartConstants'

export const addToCart = (id, qty) => async (dispatch, getState) => {
  const {data} = await axios.get(`/api/products/${id}`)
  
  dispatch({
    type:CART_ADD_ITEM,
    payload:{
      product:data._id,
      name:data.name,
      image:data.image,
      price:data.price,
      countInStock:data.countInStock,
      qty
    }
  })
  
  localStorage.setItem('cartItems', JSON.stringify(getState().cart.cartItems))
}
```

- Go to `store.js` and modify as below

```
...
const reducer = combinereducers({
  ...
})

const cartItemsFromStorage = localStorage.getItem('cartItems') ? 
  JSON.parse(localStorage.getItem('cartItems')) : []
  
const initialState = {
  cart: { cartItems: cartItemsFromStorage }
}
```

## Add to Cart Functionality
- Go to `CartScreen.js` and modify as below
- Modify the match stuff as in previous guides*

```
import React, { useEffect } from 'react'
import { Link } from 'react-router-dom'
import { useDispatch, useSelector } from 'react-redux'
import { Row, Col, ListGroup, Image, Form, Button, Card } from 'react-bootstrap'
import { Message } from '../components/Message'
import { addToCart } from '../actions/cartActions'

function CartScreen({ match, location, history }){
  const productId = match.params.id
  const qty = location.search ? Number(location.search.split('=')[1]) : 1
  
  const dispatch = useDispatch()
  
  useEffect(() => {
    if(productId){
      dispatch(addToCart(productId, qty))
    }
  }, [dispatch, productId, qty])

  return (
    <div>
      Cart
    </div>
  )
}
```
