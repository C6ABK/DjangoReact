# Frontend Authentication
## User Login Reducer & Action
- First update the error message in `productActions.js`

```
export const listProducts = () => async (dispatch) => {
  try {
    ...
  } catch (error) {
    dispatch ({
      type: PRODUCT_LIST_FAIL,
      payload: error.response && error.response.data.detail
        ? error.response.data.detail
        : error.message,
    })
  }
}

export const listProductDetails = (id) => async (dispatch) => {
  try {
    ...
  } catch (error) {
    dispatch ({
      type: PRODUCT_DETAILS_FAIL,
      payload: error.response && error.response.data.detail
        ? error.response.data.detail
        : error.message,
    })
  }
}
```

### userConstants.js
- Create `constants/userConstants.js`

```
export const USER_LOGIN_REQUEST = 'USER_LOGIN_REQUEST'
export const USER_LOGIN_SUCCESS = 'USER_LOGIN_SUCCESS'
export const USER_LOGIN_FAIL = 'USER_LOGIN_FAIL'

export const USER_LOGOUT = 'USER_LOGOUT'
```

### userReducers.js
- Create `reducers/userReducers.js`

```
import {
  USER_LOGIN_REQUEST,
  USER_LOGIN_SUCCESS,
  USER_LOGIN_FAIL,
  USER_LOGOUT,
} from '../constants/userConstants'

export const userLoginReducer = (state = {}, action) => {
  switch (action.type) {
    case USER_LOGIN_REQUEST:
      return { loading: true }

    case USER_LOGIN_SUCCESS:
      return { loading: false, userInfo: action.payload }

    case USER_LOGIN_FAIL:
      return { loading: false, error: action.payload }

    case USER_LOGOUT:
      return {}

    default:
      return state
  }
}
```

### store.js
- Modify `store.js` as below

```
...
import { userLoginReducer } from './reducers/userReducers'

const reducer = combineReducers({
  ...
  userLogin: userLoginReducer,
})
```

### userActions.js
- Create `actions/userActions.js`

```
import axios from 'axios'
import {
  USER_LOGIN_REQUEST,
  USER_LOGIN_SUCCESS,
  USER_LOGIN_FAIL,
  USER_LOGOUT,
} from '../constants/userConstants'

export const login = (email, password) => async(dispatch) => {
  try {
    dispatch({
      type: USER_LOGIN_REQUEST
    })

    const config = {
      headers: {
        'Content-type':'application/json'
      }
    }

    const {data} = await axios.post(
      '/api/users/login/'
      { 'username': email, 'password': password },
      config
    )

    dispatch({
      type: USER_LOGIN_SUCCESS,
      payload: data
    })

    localStorage.setItem('userInfo', JSON.stringify(data))

  } catch (error) {
    dispatch({
      type: USER_LOGIN_FAIL,
      payload: error.response && error.response.data.detail
        ? error.response.data.detail
        : error.message,
    })
  }
}
```

### Back to store.js
- Add `userInfoFromStorage`

```
...
const userInfoFromStorage = localStorage.getItem('userInfo') ?
  JSON.parse(localStorage.getItem('userInfo')) : null

const initialState = {
  cart: { cartItems: cartItemsFromStorage },
  userLogin: { userInfo: userInfoFromStorage }
}

...
```
