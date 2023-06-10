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
# Use this to test but remove when customising JWT Fields
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
-  Go to jwt.io to view access token information you get from the above login route

## Customise JWT Token
- Add the code below in `settings.py` below `REST_FRAMEWORK`

```
from datetime import timedelta

SIMPLE_JWT = {
  'ACCESS_TOKEN_LIFETIME': timedelta(days=30),
  'REFRESH_TOKEN_LIFETIME': timedelta(days=1),
  'ROTATE_REFRESH_TOKENS': False,
  'BLACKLIST_AFTER_ROTATION': True,
  'UPDATE_LAST_LOGIN': False,
  
  'ALGORITHM': 'HS256',
  'VERIFYING_KEY': None,
  'AUDIENCE': None,
  'ISSUER': None,
  
  'AUTH_HEADER_TYPES': ('Bearer',),
  'AUTH_HEADER_NAME': 'HTTP_AUTHORIZATION',
  'USER_ID_FIELD': 'id',
  'USER_ID_CLAIM': 'user_id',
  
  'AUTH_TOKEN_CLASSES': ('rest_framework_simplejwt.tokens.AccessToken',),
  'TOKEN_TYPE_CLAIM': 'token_type',
  
  'JTI CLAIM': 'iti',
  
  'SLIDING_TOKEN_REFRESH_EXP_CLAIM': 'refresh_exp',
  'SLIDING_TOKEN_LIFETIME': timedelta(minutes=5),
  'SLIDING_TOKEN_REFRESH_LIFETIME': timedelta(days=1),
}
```

## Customising JWT Token fields
- Go to `base/views.py` and add the following imports

```
from rest_framework_simplejwt.serializers import TokenObtainPairSerializer
from rest_framework_simplejwt.views import TokenObtainPairView
```

- Remove the previous import from `urls.py`
```
# DELETE THIS
from rest_framework_simplejwt.views import (
  TokenObtainPairView,
)
```

- Go back to `views.py` and add the following classes...

```
class MyTokenObtainPairSerializer(TokenObtainPairSerializer):
  def validate(self, attrs):
    data = super().validate(attrs)
    
    data['username'] = self.user.username
    data['email'] = self.user.email
    
    return data
    
class MyTokenObtainPairView(TokenObtainPairView):
  serializer_class= MyTokenObtainPairSerializer
```

- Go to `urls.py` and update the route as below
```
path('users/login/', views.MyTokenObtainPairView.as_view(),
```

## User Serializer
- Go to `serializers.py` and add the `UserSerializer` class

```
class UserSerializer(serializers.ModelSerializer):
  name = serializers.SerializerMethodField(read_only=True)
  _id = serializers.SerializerMethodField(read_only=True)
  isAdmin = serializers.SerializerMethodField(read_only=True)

  class Meta:
    model = User
    fields = ['id', '_id', 'username', 'email', 'name', 'isAdmin']
    
  def get__id(self, obj):
    return obj.id
    
  def get_isAdmin(self, obj):
    return obj.is_staff
    
  def get_name(self, obj):
    name = obj.first_name
    if name == '':
      name = obj.email
    
    return name
```

- Import into `base/views.py`

```
from .serializers import ProductSerializer, UserSerializer
```

- Staying in `views.py`, create a view as below

```
@api_view(['GET'])
def getUserProfile(request):
  user = request.user
  products = Product.objects.all()
  serializer = UserSerializer(user, many=False)
  return Response(serializer.data)
```

- In `base/urls.py` add the following path

```
...
path('users/profile/', views.getUserProfile, name="users-profile"),
...
```

- Test this in Postman by sending a get request to /api/users/profile/.
  - Send `Authorization` - `Bearer accesstokenhere` and it should return the user's ID, Username and Email

- In `serializers.py` ...

```
...
from rest_framework_simplejwt.tokens import RefreshToken
...
```

- Add the `UserSerializerWithToken` class

```
...
class UserSerializerWithToken(UserSerializer):
    token = serializers.SerializerMethodField(read_only=True)
    class Meta:
      model = User
      fields = ['id', '_id', 'username', 'email', 'name', 'isAdmin', 'token']
      
    def get_token(self, obj):
      token = RefreshToken.for_user(obj)
      return str(token.access_token)
    
...
```

- Go to `base/views.py` and modify as below...

```
... 
from .serializers import ProductSerializer, UserSerializer, UserSerializerWithToken

class MyTokenObtainPairSerializer(TokenObtainPairSerializer):
  def validate(self, attrs):
    data = super().validate(attrs)
    
    serializer = UserSerializerWithToken(self.user).data
    
    for k, v in serializer.items():
      data[k] = v
    
    return data
```
