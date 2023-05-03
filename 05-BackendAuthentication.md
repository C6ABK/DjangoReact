# Backend Authentication
## Postman
- In Postman, create a new collection and add the following...
  - `GET /api/products`
  - `GET /api/products/:id`
- Add the relevant URL for each and make sure the method is correct - `GET`
- Save the routes
- Create a new local variable named "URL", pass in the base url for `initial value` and `current value`
- You can now create routes in this format - `{{URL}}/api/products`
- Make sure you're serving on a public IP if using the Codeanywhere IDE - `0.0.0.0:8000`
