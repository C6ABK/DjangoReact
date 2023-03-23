# 01 - Starting the Front End
- `mkdir ecommerce` and `cd ecommerce`
- `npx create-react-app frontend`
- `cd frontend` when done.
- `npm start` to test, check running ok and enough space if on Codeanywhere IDE.
- Change title and favicon to the one in the resources folder.
- Get rid of the following files...
```
setupTests.js
App.css
App.test.js
```
- Go to `App.js` and delete everything inside the main div and remove `className="App"`

## React Bootstrap, Header & Footer Components
- Create `components` folder in the `src` directory.
- Create `Header.js` and populate with `_rfce`
- Within `Header.js`, change the `div` to `header` and enter the title Header.
- Create `Footer.js` and populate with `_rfce`
- Within `Footer.js`, change the `div` to `footer` and enter the title Footer.

### Importing Components
- Go to `App.js`
- Use `imp` for import shortcut
- `import Header from './components/Header'`
- Import the Footer to App.js as above.
- Add the components...

```
function App(){
  return (
    <div>
      <Header />
      <main>
        <h1>Welcome</h1>
      </main>
      <Footer />
    </div>
  )
}
```

### React Bootstrap
- Tutorial is using the Lux theme from bootswatch.com.
- Get the `bootstrap.min.css` file from resources and put in `src` directory.
- Go to index.js and `import './bootstrap.min.css'`
- `npm install react-bootstrap`
- Open `App.js` 
- `imd` for destructured import shortcut
- `import { Container } from 'react-bootstrap'
- Put a `Container` in the `main` section and add class to `main`

```
function App(){
  return (
    <div>
      <Header />
      <main className="py-3">
        <Container>
          <h1>Welcome</h1>
        </Container>
      </main>
      <Footer />
    </div>
  )
}
```
#### Footer.js
- Open `Footer.js`
- `imd` -> `import { Container, Row, Col } from 'react-bootstrap'
- Populate `Footer` as below

```
function Footer(){
  return (
    <footer>
      <Container>
        <Row>
          <Col className="text-center py-3">Copyright &copy; ProShop</Col>
        </Row>
      </Container>
    </footer>
  )
}
```

- Go to `index.css`
```
main {
  min-height: 80vh;
}
```

#### Header.js
- Open `Header.js` and modify as below.

```
import { Navbar, Nav, Container, Row } from 'react-bootstrap'

function Header() {
  return (
    <header>
      <Navbar bg="dark" variant="dark" expand="lg" collapseOnSelect>
        <Container>
          <Navbar.Brand href="/">ProShop</Navbar.Brand>
          <Navbar.Toggle aria-controls="basic-navbar-nav" />
          <Navbar.Collapse id="basic-navbar-nav">
            <Nav className="mr-auto">
              <Nav.Link href="/cart"><i className="fas fa-shopping-cart"></i>Cart</Nav.Link>
              <Nav.Link href="/login"><i className="fas fa-user"></i>Login</Nav.Link>
            </Nav>
          </Navbar.Collapse>
        </Container>
      </Navbar>
    </header>
  )
}
```

- Add the fontawesome CDN in `index.html`

## Home Screen Product Listing
- Move the `product.js` file from the resources into the `src` directory.
- Move the `images` folder into the `public` directory.
- Create a `screens` folder in the `src` directory.
- Create `HomeScreen.js` and populate as below using `_rfce`...

```
import { Row, Col } from 'react-bootstrap' 
import products from '../products'
import Product from '../components/Product'

function HomeScreen(){
  return (
    <div>
      <h1>Latest Products</h1>
      <Row>
        {products.map(product => (
          <Col key={product._id} sm={12} md={6} lg={4} xl={3}>
            <Product product={product} />
          </Col>
        ))}
      </Row>
    </div>
  )
}
```

- Go to `App.js`
- `import HomeScreen from './screens/HomeScreen'
- Add `<HomeScreen />` in the container and remove the welcome text.

### Product Component
- Create `Product.js` in the `components` directory
- `_rfce` to populate and fill as below

```
import { Card } from 'react-bootstrap'

function Product({ product }) {
  return (
    <Card className="my-3 p-3 rounded">
      <a href={`/product/${product._id}`}>
        <Card.Img src={product.image} />
      </a>
      
      <Card.Body>
        <a href={`/product/${product._id}`}>
          <Card.Title as="div">
            <strong>{product.name}</strong>
          </Card.Title>
        </a>
        
        <Card.Text as="div">
          <div className="my-3">
            {product.rating} from {product.numReviews} reviews
          </div>
        </Card.Text>
        
        <Card.Text as="h3">
          ${product.price}
        </Card.Text>
      </Card.Body>
    </Card>
  )
}
```

### Rating Component
- Add the component to `Product.js` to replace the existing format.

```
import Rating from './Rating'

...
<Card.Text as="div">
  <div className="my-3">
   <Rating value={product.rating} text={`${product.numReviews} reviews`} color={'#f8e825'} />
  </div>
</Card.Text>
...
```

- Create `Rating.js` in `components`
- `_rfce` to populate and fill out as below.

```
function Rating({ value, text, color }) {
  return (
    <div className="rating">
    
      <span>
        <i style={{ color }} className={
          value >= 1
            ? 'fas fa-star'
            : value >= 0.5
              ? 'fas fa-star-half-alt'
              : 'far fa-star'
        }>
        </i>
      </span>
      
       <span>
        <i style={{ color }} className={
          value >= 2
            ? 'fas fa-star'
            : value >= 1.5
              ? 'fas fa-star-half-alt'
              : 'far fa-star'
        }>
        </i>
      </span>
      
      <span>
        <i style={{ color }} className={
          value >= 3
            ? 'fas fa-star'
            : value >= 2.5
              ? 'fas fa-star-half-alt'
              : 'far fa-star'
        }>
        </i>
      </span>
      
      <span>
        <i style={{ color }} className={
          value >= 4
            ? 'fas fa-star'
            : value >= 3.5
              ? 'fas fa-star-half-alt'
              : 'far fa-star'
        }>
        </i>
      </span>
      
      <span>
        <i style={{ color }} className={
          value >= 5
            ? 'fas fa-star'
            : value >= 4.5
              ? 'fas fa-star-half-alt'
              : 'far fa-star'
        }>
        </i>
      </span>
      
      <span>{ text && text }</span>
      
    </div>
  )
}
```

- Go to `index.css`

```
.rating span {
  margin: 0.1rem;
}

h3 {
  padding: 1rem 0;
}
```

## React Router
- Make sure you're in the `frontend` directory.
- `npm install react-router-dom react-router-bootstrap`
- Add `ProductScreen.js` to screens and fill with `_rfce`
- Go to `App.js` and modify as below

```
...
import { BrowserRouter as Router, Route, Routes } from 'react-router-dom'
import ProductScreen from './screens/ProductScreen'
...

...
function App() {
  return (
    <Router>
    <Header />
    <main className="py-3">
      <Container>
        <Routes>
          <Route path="/" element={<HomeScreen />} exact />
          <Route path="/product/:id" element={<ProductScreen />} />
        </Routes>
      </Container>
    </main>
    <Footer />
    </Router>
  )
}
...

```

- Go to the `Product.js` component 
- Import the Link component `import { Link } from 'react-router-dom'`
- Change the anchor tags to `Link` tags (href becomes 'to').
```
<Link to={`/product/${product._id}`>
  <Card.Img src={product.image} />
</Link>
```
- Do the same for the product title link.
- Go to `Header.js`
- `import { LinkContainer } from 'react-router-bootstrap'`
- Wrap `<Nav.Brand>` or `<Nav.Link>` in the `<LinkContainer>` tags and remove the hrefs for each. Each link should look like this...

```
...
<Container>
  <LinkContainer to='/'>
    <Navbar.Brand>ProShop</Navbar.Brand>
  </LinkContainer>
...

...
  <LinkContainer to='/cart'>
    <Nav.Link><i className="fas fa-shopping-cart"></i>Cart</Nav.Link>
  </LinkContainer>
  <LinkContainer to='/login'>
    <Nav.Link><i className="fas fa-user"></i>Login</Nav.Link>
  </LinkContainer>
...
```

## ProductScreen.js
```
import { Link, useParams } from 'react-router-dom'
import { Row, Col, Image, ListGroup, Button, Card } from 'react-bootstrap'
import Rating from '../components/Rating'
import products from '../products'

function ProductScreen() {
  const product_id = useParams()
  const product = products.find((p) => p._id === product_id.id)
  
  return (
    <div>
      <Link to='/' className="btn btn-light my-3">Go Back</Link>
      <Row>
        <Col md={6}>
          <Image src={product.image} alt={product.name} fluid />
        </Col>
        
        <Col md={3}>
          <ListGroup variant="flush">
          
            <ListGroup.Item>
              <h3>{product.name}</h3>
            </ListGroup.Item>
            
            <ListGroup.Item>
              <Rating value={product.rating} text={`${product.numReviews} reviews`} color={"f8e825"} />
            </ListGroup.Item>
            
            <ListGroup.Item>
              Price: ${product.price}
            </ListGroup.Item>
            
            <ListGroup.Item>
              Description: {product.description}
            </ListGroup.Item>
            
          </ListGroup>
        </Col>
        
        <Col md={3}>
          <Card>
            <ListGroup variant="flush">
            
              <ListGroup.Item>
                <Row>
                  <Col>Price: </Col>
                  <Col>
                    <strong>${product.price}</strong>
                  </Col>
                </Row>
              </ListGroup.Item>
              
              <ListGroup.Item>
                <Row>
                  <Col>Status: </Col>
                  <Col>
                    {product.countInStock > 0 ? "In Stock" : "Out of Stock"}
                  </Col>
                </Row>
              </ListGroup.Item>
              
              <ListGroup.Item>
                <Button className="btn-block" disabled={product.countInStock == 0} type="button">
                  Add to Cart
                </Button>
              </ListGroup.Item>
              
            </ListGroup>
          </Card>
        </Col>
      </Row>
    </div>
  )
  
}
```
