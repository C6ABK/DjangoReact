# Backend Authentication
## Postman
- In Postman, create a new collection and add the following...
  - `GET /api/products`
  - `GET /api/products/:id`
- Add the relevant URL for each and make sure the method is correct - `GET`
- Save the routes
- Create a new local variable/Environment named "URL", pass in the base url for `initial value` and `current value`
- You can now create routes in this format - `{{URL}}/api/products`
- Make sure you're serving on a public IP if using the Codeanywhere IDE - `0.0.0.0:8000`

## JSON Web Tokens
- `pip install djangorestframework-simplejwt` in the backend directory
- Go to `settings.py` and add the code below above `MIDDLEWARE = [...]`

```
  REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
      'rest_framework_simplejwt.authentication.JWTAuthentication',
    )
  }
```

- Check docs for different token types (refresh tokens), here we will just change the token lifespan...
- Go to `base/urls.py` and edit as below...

```
...
from rest_framework_simplejwt.views import (
  TokenObtainPairView,
)

# Add at the top of urlpatterns
# Check if you need api/ if you've used proxy
urlpatterns = [
  path('api/users/login/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
  ...
]

```
