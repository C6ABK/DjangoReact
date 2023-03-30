# Adding to Cart
## Qty Select
- Open `ProductScreen.js` and modify as below...
  - Edit the `match` section in this document later

```
...
import { Row, Col, Image, ListGroup, Button, Card, Form } from 'react-bootstrap'
...

function ProductScreen({ match, history }) {
  const [qty, setQty] = useState(1)
 
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
  history.push(`/cart/${match.params.id}?qty=${qty}`)
}
...
```

- Add `CartScreen.js` in the `screens` directory and fill with `rfce`
