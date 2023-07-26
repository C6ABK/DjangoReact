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

## Login Screen & Functionality
- Create `screens/LoginScreen.js

```
import React, { useState, useEffect } from 'react'
import { Link } from 'react-router-dom'
import { Form, Button, Row, Col } from 'react-bootstrap'
import { useDispatch, useSelector { from 'react-redux'
import Loader from '../components/Loader'
import Message from '../components/Message'
import { login } from '../actions/userActions'

function LoginScreen(){
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')

  return (
    <div>
      Login Screen
    </div>
  )
}

export default LoginScreen
```

- Go to `App.js` and add the route

```
...
import LoginScreen from './screens/LoginScreen'

function App(){
  return (
    <Header />
    <main className="py-3">
      <Container>
        <Route path="/" component={HomeScreen} exact />
        <Route path="/login" component={LoginScreen} />
        ...
      </Container>
    </main>
  )
}
```

### Form Container
- Create `components/FormContainer.js`

```
import React from 'react'
import { Container, Row, Col } from 'react-bootstrap'

function FormContainer({ children }) {
  return (
    <Container>
      <Row className="justify-content-md-center">
        <Col xs={12} md={6}>
          {children}
        </Col>
      </Row>
    </Container>
  )
}
```

- Go back to `LoginScreen.js` and modify as below to import the form container

```
...
import FormContainer from '../components/FormContainer'
import { login } from '../actions/userActions'
...

function LoginScreen({location, history}){
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')

  const dispatch = useDispatch()

  const redirect = location.search ? location.search.split('=')[1] : '/'

  const userLogin = useSelector(state => state.userLogin)
  const { error, loading, userInfo } = userLogin

  useEffect(() => {
    if (userInfo) {
      history.push(redirect)
    }
  }, [history, userInfo, redirect])

  const submitHandler = (e) => {
    e.preventDefault()
    dispatch(login(email, password))
  }

  return (
    <FormContainer>
      <h1>Sign In</h1>
      {error && <Message variant='danger'>{error}</Message>}
      {loading && <Loader />}

      <Form onSubmit={submitHandler}>

        <Form.Group controlId='email'>
          <Form.Label>Email Address</Form.Label>
          <Form.Control
            type='email'
            placeholder='Enter Email'
            value={email}
            onChange={(e) => setEmail(e.target.value)}
          >
          </Form.Control>
        </Form.Group>

        <Form.Group controlId='password'>
          <Form.Label>Password</Form.Label>
          <Form.Control
            type='password'
            placeholder='Enter Password'
            value={password}
            onChange={(e) => setPassword(e.target.value)}
          >
          </Form.Control>
        </Form.Group>

        <Button type='submit' variant='primary'>
          Sign In
        </Button>

      </Form>

      <Row className='py-3'>
        <Col>
          New Customer?
          <Link to={redirect ? '/register?redirect=${redirect}' : '/register'}>
            Register
          </Link>
        </Col>
      </Row>
    </FormContainer>
  )
}

export default LoginScreen
```

## User in Navbar & Logout
- Open `components/Header.js` and modify as below

```
...
import { useDispatch, useSelector } from 'react-redux'
import { logout } from '../actions/userActions'
...

function Header() {
  const userLogin = useSelector(state => state.userLogin)
  const { userInfo } = userLogin

const dispatch = useDispatch()

  const logoutHandler = () => {
    dispatch(logout())
  }

  return (
    <header>
      ...
      <Navbar.Collapse id="basic-navbar-nav">
        <Nav className="mr-auto">
          <LinkContainer to='/cart'>
            <Nav.Link>...</Nav.Link>
          </LinkContainer

          {userInfo ? (
            <NavDropdown title={userInfo.name} id='username'>
              <LinkContainer to='/profile'>
                <NavDropdown.Item>Profile</NavDropdown.Item>
              </LinkContainer>
              <NavDropdown.Item onClick={logoutHandler}>Logout</NavDropdown.Item>
            </NavDropdown>
          ): (
            <LinkContainer to='/login'>
              <Nav.Link>...</Nav.Link>
            </LinkContainer
          )}
        </Nav>
      </Navbar.Collapse>
    </header>
  )
}
```

- Go to `actions/userActions.js` and add the logout at the bottom

```
...
export const logout = () => (dispatch) => {
  localStorage.removeItem('userInfo')
  dispatch(type: USER_LOGOUT)
}
```

## User Register Reducer, Action & Screen
- Go into `constants/userConstants.js` and modify as below

```
...
export const USER_REGISTER_REQUEST = 'USER_REGISTER_REQUEST'
export const USER_REGISTER_SUCCESS = 'USER_REGISTER_SUCCESS'
export const USER_REGISTER_FAIL = 'USER_REGISTER_FAIL'
```

- Go to `reducers/userReducers.js` and modify as below

```
import {
  ...
  USER_REGISTER_REQUEST,
  USER_REGISTER_SUCCESS,
  USER_REGISTER_FAIL,
} from '../constants/userConstants'

...

export const userRegisterReducer = (state = {}, action) => {
  switch (action.type) {
    case USER_REGISTER_REQUEST:
      return { loading: true }

    case USER_REGISTER_SUCCESS:
      return { loading: false, userInfo: action.payload }

    case USER_REGISTER_FAIL:
      return { loading: false, error: action.payload }

    case USER_LOGOUT:
      return {}
  }
}
```

- Go to `store.js`

```
...
import { userLoginReducer, userRegisterReducer } from './reducers/userReducers'

const reducer = combineReducers({
  ...
  userLogin: userLoginReducer,
  userRegister: userRegisterReducer,
})
```

- Go to `actions/userActions.js`

```
...
import {
  ...
  USER_REGISTER_REQUEST,
  USER_REGISTER_SUCCESS,
  USER_REGISTER_FAIL,  
} from '../constants/userConstants'

...

export const register = (name, email, password) => async (dispatch) => {
  try {
    dispatch({
      type: USER_REGISTER_REQUEST
    })

    const config = {
      headers: {
        'Content-type': 'application/json'
      }
    }

    const { data } = await axios.post(
      '/api/users/register/',
      { 'name': name, 'email': email, 'password': password },
      config
    )

    dispatch({
      type: USER_REGISTER_SUCCESS,
      payload: data
    })

    dispatch({
      type: USER_LOGIN_SUCCESS,
      payload: data
    })

    localStorage.setItem('userInfo', JSON.stringify(data))

  } catch (error) {
    dispatch({
      type: USER_REGISTER_FAIL,
      payload: error.response && error.response.data.detail
    })
  }
}
```

- Create `screens/RegisterScreen.js`

```
import React, { useState, useEffect } from 'react'
import { Link } from 'react-router-dom'
import { Form, Button, Row, Col } from 'react-bootstrap'
import { useDispatch, useSelector } from 'react-redux'
import Loader from '../components/Loader'
import Message from '../components/Message'
import FormContainer from '../components/FormContainer'
import { login } from '../actions/userActions'

function RegisterScreen({ location, history }) {

  const [name, setName] = useState('')
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')
  const [confirmPassword, setConfirmPassword] = useState('')
  const [message, setMessage] = useState('')

  const dispatch = useDispatch()

  const redirect = location.search ? location.search.split('=')[1] : '/'

  const userRegister = useSelector(state => state.userRegister)
  const { error, loading, userInfo } = userRegister

  useEffect(() => {
    if (userInfo) {
      history.push(redirect)
    }
  }, [history, userInfo, redirect])

  const submitHandler = (e) => {
    e.preventDefault()

    if (password != confirmPassword){
      setMessage('Passwords do not match')
    } else {
      dispatch(register(name, email, password))
    }
  }

  return (
    <FormContainer>
      <h1>Register</h1>
      {message && <Message variant='danger'>{message}</Message>}
      {error && <Message variant='danger'>{error}</Message>}
      {loading && <Loader />}
      <Form onSubmit={submitHandler}>

        <Form.Group controlId='name'>
          <Form.Label>Name</Form.Label>
          <Form.Control
            required
            type='name'
            placeholder='Enter Name'
            value={name}
            onChange={(e) => setName(e.target.value)}
          >
          </Form.Control>
        </Form.Group>

        <Form.Group controlId='email'>
          <Form.Label>Email Address</Form.Label>
          <Form.Control
            required
            type='email'
            placeholder='Enter Email'
            value={email}
            onChange={(e) => setEmail(e.target.value)}
          >
          </Form.Control>
        </Form.Group>

        <Form.Group controlId='password'>
          <Form.Label>Password</Form.Label>
          <Form.Control
            required
            type='password'
            placeholder='Enter Password'
            value={password}
            onChange={(e) => setPassword(e.target.value)}
          >
          </Form.Control>
        </Form.Group>

        <Form.Group controlId='passwordConfirm'>
          <Form.Label>Confirm Password</Form.Label>
          <Form.Control
            required
            type='password'
            placeholder='Confirm Password'
            value={confirmPassword}
            onChange={(e) => setConfirmPassword(e.target.value)}
          >
          </Form.Control>
        </Form.Group>

        <Button type='submit' variant='primary'>
          Register
        </Button>

      </Form>

      <Row className='py-3'>
        <Col>
          Have an Account?
          <Link
            to={redirect ? `/login?redirect=&{redirect}` : '/login'}
          >
            Sign In
          </Link>
        </Col>
      </Row>
    </FormContainer>
  )
}
export default RegisterScreen
```

- Go to `App.js` and modify as below

```
...
import RegisterScreen from './screens/RegisterScreen'

...

<Container>
  ...
  <Route path='/register' component={RegisterScreen} />
</Container>

...
```

## Update Profile Endpoint

- Go to `backend/base/views/user_views.py` and add the API view as below

```
...
@api_view(['PUT'])
@permission_classes([IsAuthenticated])
def updateUserProfile(request):
  user = request.user
  serializer = UserSerializerWithToken(user, many=False)

  data = request.data

  user.first_name = data['name']
  user.username = data['email']
  user.email = data['email']

  if data['password'] != '':
    user.password = make_password(data['password'])

  user.save()

  return Response(serializer.data)
  
...
```

- Go to `backend/base/urls/user_urls.py`

```
...
path('profile/update/', views.updateUserProfile, name="user-profile-update"),
...
```

- Go to `backend.serializers.py` and update the `UserSerializerWithToken(UserSerializer)` as below to get an access token

```
class UserSerializerWithToken(UserSerializer):
  token = serializers.SerializerMethodField(read_only=True)

  class Meta:
    model = User
    fields = ['id', '_id', 'username', 'email', 'name', 'isAdmin', 'token']

  def get_token(self, obj):
    token = RegreshToken.for_user(obj)
    return str(token.access_token)
```
