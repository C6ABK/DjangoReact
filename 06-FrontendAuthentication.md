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
