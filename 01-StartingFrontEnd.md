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
- Go to index.js and `import './bootstrap.min.css`
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
import { Navbbar, Nav, Container, Row } from 'react-bootstrap'

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
