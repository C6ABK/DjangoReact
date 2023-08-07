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

## Protected Routes
- Go to `base/views.py` and import the `permission_classes` decorator and permissions

```
...
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAuthenticated, IsAdminUser
...
```

- Modify the user profile route as below

```
@api_view(['GET'])
@permission_classes([IsAuthenticated])
def getUserProfile(request):
  ...
```

- Create a user's route that should only be visible for an admin user

```
from django.contrib.auth.models import User

...

@api_view(['GET'])
@permission_classes([IsAdminUser])
def getUsers(request):
  users = User.objects.all()
  serializer = UserSerializer(users, many=True)
  return Response(serializer.data)
```

- Go to `base/urls.py` and add the users route

```
...
path('users/', views.getUsers, name="users"),
...
```

## Register User
- Go to `base/views.py` and get rid of `def getRoutes`
- Go to `base/urls.py` and get rid of the `view.getRoutes` path
- Go back to `base/views.py` and create the `registerUser` route as below

```
from django.contrib.auth.hashers import make_password
from rest_framework import status

...

@api_view(['POST'])
def registerUser(request):
  data = request.data

  try:
    user = User.objects.create(
      first_name=data['name'],
      username=data['email'],
      email=data['email'],
      password=make_password(data['password']),
    )
    serializer = UserSerializerWithToken(user, many=False)
    return Response(serializer.data)
  except:
    message = {'detail':'User with this email already exists'}
    return Response(message, status=status.HTTP_400_BAD_REQUEST)
```

- Go to `base/urls.py` and add the path
```
...
path('users/register/', views.registerUser, name='register'),
...
```

## Login With Email

### Django Signals
- Signals allow certain senders to notify a set of receivers that some action has taken place.
- We want to use pre_save and post_save here
- Go to `base` and create `signals.py`
- Go to `base/apps.py` and modify as below
- Ensure `settings.py` contains `'base.apps.BaseConfig',`

```
from django.apps import AppConfig

class BaseConfig(AppConfig):
  name = 'base'

  def ready(self):
    import base.signals
```

- Fill `signals.py` as below to test config

```
from django.db.models.signals import pre_save
from django.contrib.auth.models import User

def updateUser(sender, instance, **kwargs):
  print('Signal triggered')

pre_save.connect(updateUser, sender=User)
```

- Modify `signals.py` as below

```
from django.db.models.signals import pre_save
from django.contrib.auth.models import User

def updateUser(sender, instance, **kwargs):
  user = instance 
  if user.email != '':
    user.username = user.email

pre_save.connect(updateUser, sender=User)
```

## URLs and Views Cleanup
- Go to `backend/urls.py` and modify the `urlpatterns` as below

```
urlpatterns = [
  path('admin/', admin.site.urls),
  #path('api/', include('base.urls')), delete/comment out this line
  path('api/products/', include('base.urls.product_urls')),
  path('api/users/', include('base.urls.user_urls')),
  path('api/orders/', include('base.urls.order_urls')),
]
```

- Go to `base` and create the `views` folder
- In `base/views` create the following files
  - order_views.py
  - product_views.py
  - user_views.py

### product_views.py

```
from django.shortcuts import render
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAuthenticated, IsAdminUser
from rest_framework.response import Response
from base.models import Product
from base.serializers import ProductSerializer
from rest_framework import status

@api_view(['GET'])
def getProducts(request):
  product = Product.objects.all()
  serializer = ProductSerializer(products, many=True)
  return Response(serializer.data)

@api_view(['GET'])
def getProduct(request, pk):
  product = Product.objects.get(_id=pk)
  serializer = ProductSerializer(product, many=False)
  return Response(serializer.data)
```

### user_views.py

```
from django.shortcuts import render
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAuthenticated, IsAdminUser
from rest_framework.response import Response
from base.serializers import ProductSerializer, UserSerializer, UserSerializerWithToken
from rest_framework_simplejwt.serializers import TokenObtainPairSerializer
from rest_framework_simplejwt.views import TokenObtainPairView
from django.contrib.auth.hashers import make_password
from rest_framework import status

class MyTokenObtainPairSerializer(TokenObtainPairSerializer):
  def validate(self, attrs):
    data = super().validate(attrs)

    serializer = UserSerializerWithToken(self.user).data
    for k, v in serializer.items():
      data[k] = v

    return data

class MyTokenObtainPairView(TokenObtainPairView):
  serializer_class = MyTokenObtainPairSerializer

@api_view(['POST'])
def registerUser(request):
  data = request.data

  try:
    user = User.objects.create(
      first_name=data['name'],
      username=data['email'],
      email=data['email'],
      password=make_password(data['password'])
    )

    serializer = UserSerializerWithToken(user, many=False)
    return Response(serializer.data)
  except:
    message = {'detail':'User with this email already exists'}
    return Response(message, status=status.HTTP_400_BAD_REQUEST)

@api_view(['GET'])
@permission_classes([IsAuthenticated])
def getUserProfile(request):
  user = request.user
  serializer = UserSerializer(user, many=False)
  return Response(serializer.data)

@api_view(['GET'])
@permission_classes([IsAdminUser])
def getUsers(request):
  users = User.objects.all()
  serializer = UserSerializer(users, many=True)
  return Response(serializer.data)
```

- `base/views.py` can now be deleted
- Create the `urls` folder in `base`
- Create the following in `base/urls/`
  - order_urls.py
  - product_urls.py
  - user_urls.py

### product_urls.py

```
from django.urls import path
from base.views import product_views as views

urlpatterns = [
  path('', views.getProducts, name="products"),
  path('<str:pk>/', views.getProduct, name="product"),
]
```

### user_urls.py

```
from django.urls import path
from base.views import user_views as views

urlpatterns = [
  path('login/', views.MyTokenObtainPairView.as_view(), name="token_obtain_pair"),
  path('register/', views.registerUser, name="register"),
  path('profile/', views.getUserProfile, name="users-profile"),
  path('', views.getUsers, name="users"),
]
```

### order_urls.py

```
from django.urls import path
from base.views import order_views as views

urlpatterns = [

]
```

- Delete the FILE `base/urls.py`
